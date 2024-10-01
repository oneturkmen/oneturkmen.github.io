---
layout: post
title: "(In)efficient Insertions in Postgres"
date: 2024-03-16 22:21:31 -0700
description: Multiple ways to insert large amount of data in the database. 
tags: databases performance storage
categories: tech
published: true
---


PostgreSQL is a popular relational database that is used for variety of applications.
With the recent surge in popularity of Generative AI (Large Language Models in particular), PostgreSQL is often used for storing document "embeddings" (low-level representation of documents) to enhance search functionality.
In addition to the ability to "search" similar documents in a query, it is used primarily due to its reliability, speed, and community support (I know, some folks might say relational databases are slow, but there are plenty of ways to tune the database to all kinds of use cases and access patterns).

## Writing to PostgreSQL

The SQL in PostgreSQL is already a spoiler for what language is used to retrieve or store the data.
One of the common ways, which is also part of SQL standard, is to use `INSERT` command:

```sql
INSERT INTO customer (id, name, age)
VALUES (123, "john doe", 29);
```

In the example below, we insert (store) a single row containing 3 columns.
This classic usage of `INSERT` is quite common when you need to store a few
rows (e.g. when you need to insert new information in real time).

But does `INSERT` still work with larger amount of data? What if we write 100s of columns?
One way is to loop over your data records and insert them one by one:

```python
import psycopg

customers = [( 123, "john doe", 29 ), ( 125, "jane doe", 31 )]

# Connect to an existing database
with psycopg.connect("dbname=marketplace user=mytestusername") as conn:

    # Open a cursor to perform database operations
    with conn.cursor() as cur:

        # Pass data to fill a query placeholders and let Psycopg perform
        # the correct conversion (no SQL injections!)
	for id, name, age in customers:
		cur.execute(
		    "INSERT INTO customer (id, name, age) VALUES (%s, %s, %s)", (id, name, age)
		)

        # Make the changes to the database persistent
        conn.commit()
```

This works fine for a handful of queries. Once you get into 1000s or even more records,
the code above will progressively start getting worse. It is not scalable.

Another option is to use `cur.executemany()` where the same query will be executed for a list of tuples
instead of a single tuple. It is better than serially executing `INSERTS` because the transaction is committed
only once instead of multiple times. However, `executemany()` will still be very slow because it is not
optimized for very large amounts of data (>=1 million rows).

Best option for "batch" loading (when we load large amount of data at once) is to use `COPY`.
`COPY` is done only within a single transaction and treats the entire file as an input stream.
It is [specifically optimized](https://github.com/postgres/postgres/blob/c4d5cb71d229095a39fda1121a75ee40e6069a2a/src/backend/commands/copyfrom.c#L640) for batch loading.

```sql
COPY customer FROM 'customer_data.csv' DELIMITER ',';
```

## Benchmarking

I created a [repository with benchmarking code](https://github.com/oneturkmen/slow-fast-postgres-insertions) that shows the performance
difference between the three methods (i.e. `execute`, `executemany`, and `copy`).
Feel free to fork it and reproduce locally. The results might differ, but only slightly. The speed-up
ratios between the approach should stay about the same.

Here are the results from a single run:

```
Function 'very_slow_insert' [execute one by one] executed in 4309.07 secs / 71.82 mins
Function 'slow_insert' [executemany] executed in 168.56 secs / 2.81 mins
Function 'fast_copy' [copy] executed in 61.26 secs / 1.02 mins
```

`COPY` command was about **~71x** faster than running `INSERT` in a loop and **~2.8x** faster than
running a single-batch `INSERT` command.


## Conclusion

It is clear that `COPY` is the best option for large data loading in Postgres.
The single-batch `INSERT` (with many values) works too, but is still behind the copy.
The individual `INSERT` commands are the slowest and I do not recommend running them
unless you are only inserting a handful of rows and do not generally have a large load.
