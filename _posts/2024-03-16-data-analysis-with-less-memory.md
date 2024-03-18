---
layout: post
title: "Biting Off More Than We Can Chew with OLAP Libraries"
date: 2024-03-16 22:21:31 -0700
description: Comparing three popular libraries for analyzing data that is larger than available memory. 
tags: olap data-analysis python performance
categories: tech
---

### Or how to do data analysis with little memory using three popular libraries: Polars, DuckDB, and Dask!

With the [exponentially increasing volume of data](https://www.statista.com/statistics/871513/worldwide-data-created/), it would be nice to have the ability to read data larger than the available memory. 
Inspired by Daniel Beach's post on [DuckDB vs Polars](https://dataengineeringcentral.substack.com/p/duckdb-vs-polars-thunderdome), I would like to do a similar analysis
of data processing libraries that focus on high performance. The only difference is that I would not be
reading data from a cloud storage like S3. Instead, I would have the data downloaded locally on my computer. 

## Setup

**Data.** I will use [the same source of raw HD test dataset](https://www.backblaze.com/cloud-storage/resources/hard-drive-test-data#w-tabs-2-data-w-pane-1) that Daniel used (Backblaze hard drive data stats such as models, capacities, and failures). I could not determine what files exactly he downloaded, but I will use 2023 Q1, Q2, and Q3 data for our
setup. So we have a total of `~24 GB` of CSV files.

{% include figure.html path="assets/img/olap_dataset.png" class="img-fluid rounded z-depth-1" %}

**Compute.** For the compute power, I have **11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz** with 4 cores (2 threads per core; so total **8 threads**) with **16 GB of RAM**.

**Task.** Our task will be to compute the number of `failures` grouped by `date`s in the set of `~24 GB` of CSV files. In SQL, it looks like:

```sql
SELECT date, SUM(failure) as failures
FROM table_with_all_failures 
GROUP BY date
```

## Setting up the tools

**Polars.** Polars is a high-performance data processing library. It
can be used to manipulate structured data in a very fast way. While the core of the library is written in Rust, the library has APIs in Python, R, and NodeJS. Basically, think of it as a very fast alternative to [Pandas](https://pandas.pydata.org/) (but remember, it's not quite a drop-in replacement due to [some major differences](https://docs.pola.rs/user-guide/migration/pandas/) between the two).

Our test code using Polars looks like this:

```python
import polars as pl
import time

# Uncomment line below for more logs.
# pl.Config.set_verbose(True)

def polars_test():
    lazy_df = pl.scan_csv("*/*.csv")

    sql = pl.SQLContext()
    sql.register("harddrives", lazy_df)   
    results = sql.execute("""
        SELECT date, SUM(failure) as failures
        FROM harddrives 
        GROUP BY date
    """)

    results_filename = "results_polars.csv"
    results.collect().write_csv(results_filename)

start_time = time.time()
polars_test()
end_time = time.time()
print(f"It took {end_time - start_time} seconds to run Polars test.")
```

In the code snippet above, we are lazily scanning the set of CSV files into a polars `LazyFrame`, register the dataframe
as a (quasi) SQL table so that we can run the aforementioned SQL query on it. Once we get the results of the grouping
operation, we write them to a CSV file `results_polars.csv`.

**DuckDB.** DuckDB is a "fast in-process analytical database". Think of it as in-memory database that allows you
to perform very fast computations on columns (a.k.a. [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing)).
Similar to Polars, it supports a SQL dialect that can be used to query and manipulate data.

Our test code using DuckDB looks like this:

```python
import duckdb
import time


def duckdb_test():
    duckdb.sql("""
        SET memory_limit = '10GB';
        CREATE VIEW metrics AS 
        SELECT date, SUM(failure) as failures
        FROM read_csv('*/*.csv', union_by_name = true)
        GROUP BY date;
    """)

    duckdb.sql("""
        COPY metrics TO 'results_duckdb.csv';
    """)

start_time = time.time()
duckdb_test()
end_time = time.time()
print(f"It took {end_time - start_time} seconds to run DuckDB test.")
```

In the snippet above, we read the CSV files within the SQL statement using `read_csv()` function. I had to set `union_by_name`
parameter to make it work; otherwise, it throws `duckdb.duckdb.InvalidInputException: Invalid Input Error: Mismatch between the schema of different files`. 

You probably also noticed that I set `memory_limit` to `10 GB`, and that is for a good (take a guess :P) reason. I will explain it in the next section (Results).

**Dask.** Dask is a library for parallel computing in Python. It is a feature-rich library that lets you scale Python code from a single computer to large distributed clusters.

Here is our setup with Dask:

```python
import dask.dataframe as dd 
import time


def dask_test():
    dfs = dd.read_csv("*/*.csv", dtype={'failure': 'float64'})
    result_df = dfs.groupby("date").failure.sum()
    result_df.to_csv("results_dask.csv", single_file=True)

start_time = time.time()
dask_test()
end_time = time.time()
print(f"It took {end_time - start_time} seconds to run DuckDB test.")
```

The code above reads the CSV files into Dask DataFrame, groups data by `date` and computes the `sum` of `failure`s, saving
the results in another CSV file. Note that I had to cast `failure` column to `float64` because it would throw `ValueError`
recommending that I change the type from `int64` to `float64`, even though I never specified `int64`. It is most likely
that most of the entries in the column are indeed of type `int64`, however it recommends `float64` due to the presence of 
`NaN`s.

## Results

We are finally in the results section! So here are they for each of tool set up above.

**Polars.** Polars is a winner with its setup script taking between **8-15 seconds** to get the CSV file with the results.
The setup, as you have seen, looks simple enough, it is readable, no surprises.

**DuckDB.** It took between **115-140 seconds**, about **~10x** slower than Polars. However, I did not get the results on the first try. My Python process got killed a few times before
I added `SET memory_limit = '10GB';`. Why `10 GB`, you might be wondering? I am not 100% sure, I would answer,
although there is a brief explanation from [its documentation](https://duckdb.org/docs/configuration/pragmas#memory-limit
):

> The specified memory limit is only applied to the buffer manager. For most queries, the buffer manager handles the majority of the data processed. However, certain in-memory data structures such as vectors and query results are allocated outside of the buffer manager. Additionally, aggregate functions with complex state (e.g., list, mode, quantile, string_agg, and approx functions) use  memory outside of the buffer manager. Therefore, the actual memory consumption can be higher  than the specified memory limit.

It would be great to know how much DuckDB allocates additional memory for the mentioned operations. It would be even better if DuckDB had a parameter to control how much memory goes into those operations.

**Dask.** Our Dask test took between **85-95** seconds, slightly faster than the DuckDB test but slower from the Polars' one.
There were no surprises with Dask, but I also think it was not built for a single computer data processing. Even though I have not tested it yet, it would most likely shine in distributed computing environments where one can run data-intensive programs across many computers.

## Conclusion

So here we have the results for the three tools. Polars is a clear winner here, but bear in mind that I provided
results for one specific test which is about performing data analysis on data that is larger than memory. I am sure
DuckDB is as performant as Polars, although it is not clear how to handle the out-of-memory issues and how to correctly
configure memory limits. Dask is in between, likely a choice better for distributed computing environments rather
than a single computer case.
