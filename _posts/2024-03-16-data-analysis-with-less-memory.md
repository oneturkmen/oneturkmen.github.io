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

**Data.** I will use [the same source of raw HD test dataset](https://www.backblaze.com/cloud-storage/resources/hard-drive-test-data#w-tabs-2-data-w-pane-1) that Daniel used (Backblaze hard drive data stats such as models, capacities, and failures). I could not determine what files exactly he downloaded, but I will use 2022 Q1 and Q2, and all of 2023
data for our setup. So we have a total of `~45 GB` of CSV files.

```bash
$ du -sh data_* --total

6.2G	data_Q1_2022
7.1G	data_Q1_2023
6.4G	data_Q2_2022
7.6G	data_Q2_2023
8.6G	data_Q3_2023
9.0G	data_Q4_2023
45G	total
```

**Compute.** For the compute power, I have **11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz** with 4 cores (2 threads per core; so total **8 threads**). I have 16 GB RAM total, but I will limit it to **4 GB only** using `ulimit -v 4000000` (4 million bytes) for each test. 

**Task.** Our task will be to compute the number of `failures` grouped by `date`s in the set of `~45 GB` of CSV files. In SQL, it looks like:

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
    results.collect(streaming=True).write_csv(results_filename)

start_time = time.time()
polars_test()
end_time = time.time()
print(f"It took {end_time - start_time} seconds to run Polars test.")
```

In the code snippet above, we are lazily scanning the set of CSV files into a polars `LazyFrame`, register the dataframe
as a (quasi) SQL table so that we can run the aforementioned SQL query on it. Note the `.collect(streaming=True)` part
with the `streaming` parameter: it will process the data [in chunks](https://docs.pola.rs/user-guide/concepts/streaming/) because our dataset is larger than available memory. Once we get the results of the grouping
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
        SET preserve_insertion_order = false;
        SET temp_directory = './temp';

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

Also note couple configuration parameters, namely `preserve_insertion_order = false` and `temp_directory = './temp'`.
The former is set to flag DuckDB that it does not have to preserve the order of the files it reads. Disabling
the insertion order preserves memory. For the latter, setting `temp_directory` should have enabled us processing
data larger than memory. According to DuckDB, "if DuckDB is running in in-memory mode, it cannot use disk to offload data if it does not fit into main memory. To enable offloading in the absence of a persistent database file, use the `SET temp_directory` statement". Despite many tries with different parameters, I could not make it work. Some folks
needed to set [number of threads to 1](https://github.com/duckdb/duckdb/issues/11054) to make it work. Others
recommended using some [nightly build](https://github.com/duckdb/duckdb/issues/11054#issuecomment-1985758719) that fixes the issue, but it looks like the issue is still there.   

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

**DuckDB.** Unfortunately, DuckDB kept throwing "Out Of Memory" exceptions:

```bash
duckdb.duckdb.OutOfMemoryException: Out of Memory Error: Failed to allocate block of 32002048 bytes
```

As mentioned in the Setup section, I tried playing with several configuration parameters like `temp_directory` and `preserve_insertion_order`. Neither of them fixed the issue.

**Dask.** Our Dask test took between **175** seconds (~3 minutes), much slower than Polars.
There were no surprises with Dask, but I also think it was not built for single-computer data processing. Even though I have not tested it yet, it would most likely shine in distributed computing environments where one can run data-intensive programs across many computers.

## Conclusion

So here we have the results for the three tools. Polars is a clear winner here, but bear in mind that I provided
results for one specific test which is about performing data analysis on data that is larger than memory. I am sure
DuckDB is as performant as Polars, although I could not get the out-of-memory issue resolved. Dask is in between, likely a choice better for distributed computing environments rather
than a single computer case.