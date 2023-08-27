---
layout: post
title: "The Art of Controlled Evolution: How Migration Tools Shape Database Schemas"
date: 2023-04-02 21:00:00 -0700
description: Guide to managing database changes with confidence. 
tags: databases postgresql git
categories: tech
---

## Evolving requirements, evolving data 

Today's world is dynamic and chaotic making it nearly impossible to predict
what is expected in future. Similar behavior can be observed in software development,
where requirements may drastically change within days even with careful, long-term planning.
That is perhaps why Agile methodology is a popular and the "standard" approach
to software development in today's world.

Imagine a scenario in which you talk to a client about some cool mobile app
idea, and they ask you to store, process, and display some data in the app.
However, next week, they ask you us for something else, so you need to address the
change in client's requirements on data. As a consequence, you need to make
changes to the application and the way it stores, processes, and renders the data.

If we had something hard-coded in code that needed to be changed, it would be
simply changing the value of the associated structures (e.g., variables or
lists of strings). However, software developers detach code from data, and typically use storage systems for storing the data.

There is a variety of database systems, such as relational and
non-relational (e.g., NoSQL). The relational systems are great at
establishing "relations" between different data entities, which
provide safeguards against data anomalies and often make it efficient to retrieve
related data together (e.g., through table joins and column indexes).

We know how developers commonly "version" source code changes to keep track of
where the source code was and where it is at the moment. Additionally, it is important
for effective collaboration with other developers.
However, changing database schema is not as clear cut as changing code.
Accidentally modifying (or even worse, deleting data) can result in large
negative impact, including loss of monetary value and/or reputation.

Luckily, we are not the first ones to encounter such a problem.
In this post, I would like to focus on changing the schema in the
relational SQL databases, such as PostgreSQL, in effective ways
that keeps the database state consistent.
By the end of this blog post, we will learn how to effectively manage changes in SQL
databases.

## Schema changes as migrations

Software requirements change frequently, and so do database schema associated
with them. Changing the schema is a tricky task that can easily result in data
inconsistency and even unexpected downtimes. Not surprisingly, making changes
to the schema can be a nerve-wracking experience.

If schema changes are important, then we should monitor them. We should
track who makes the change, when it was made, and why it was made. We should have
something similar to
[git workflow](https://www.atlassian.com/git/tutorials/comparing-workflows), where
incremental changes to code are noted in history.

Akin to `git` commits, that keep track of changes in the application code,
schema migrations are changes to the file(s) that represent the schema.
Those changes can be applied incrementally (on top of each other) and can be
easily reverted, if something goes wrong.

Take a look at the example code snippet below where we want to split one column into two.
The initial version contained a column `TEXT full_name` that stored a person's full name.
A change to the schema was made to split `full_name` into `first_name` and `last_name`
columns.

```diff
CREATE TABLE contact (
	id INT GENERATED ALWAYS AS IDENTITY,
-	TEXT full_name
+	TEXT first_name
+	TEXT last_name
	TEXT email
);
```

The changes made to the above SQL file can be tracked using `git`. That is an incomplete
way of applying changes to the schema because they are not immediately
reflected in the database. Yes, the file changed in the `git` repository, but
that file cannot just be loaded in the database to apply the changes.
So, how do we apply such a change in the database to actually split the column
into two?

## Migration tools

To apply the changes in the database, we
can use a *SQL schema migration tool*, also known as
[database-as-code migration tool](https://www.bytebase.com/blog/top-database-schema-change-tool-evolution/#gitops-database-as-code).
There are plenty of amazing tools, both open source and free, as well as proprietary ones.

I am going to use [Alembic](https://alembic.sqlalchemy.org/en/latest/tutorial.html#the-migration-environment), a lightweight schema migration tool that uses [SQLAlchemy](https://www.sqlalchemy.org/) as its engine.

> If you have never worked with SQLAlchemy, it is a cool library that provides both object-relational mapping (ORM) and database toolkit for Python applications. It helps map SQL schema onto Python data structures for easier, in-code data management.

Alembic keeps track of changes to the database schema through *revision scripts*.
The revision scripts contain the "delta", or the change that is applied to the schema.
Let's create an example script to split a single column `full_name`
into two columns `first_name` and `last_name`.
To create a revision script, we can run the following command:

```bash
alembic revision -m "split full_name into first_name and last_name"
```

The command will generate a file named something like `a1829f4e7900_split_full_name.py`.
Note the prefix of the file name - that's a revision hash used to mark a
schema change, similar to a git commit hash. The contents of the file may look
like this:

```python
"""Split full_name into first_name and last_name

Revision ID: a1829f4e7900
Revises:
Create Date: 2023-02-02 11:40:27.089406
"""

# revision identifiers, used by Alembic.
revision = 'a1829f4e7900'
down_revision = None
branch_labels = None

from alembic import op
import sqlalchemy as sa

def upgrade():
    pass

def downgrade():
    pass
```

The file comes with a docstring with the short description of the change,
revision ID, and creation date. Note that the comment also mentions `"Revises: "`,
which indicates the previous revision ID. It is obviously empty in our file
because we just created our first revision. If we created another revision in addition
to the one we just generated, the new revision script would have `"Revises: a1829f4e7900"`.
Further, the variable `down_revision` indicates the same thing.

Note that we also have two empty functions generated for us, namely `upgrade()` and `downgrade()`. The former allows us to add logic for the new schema change, while
the latter lets us add the logic to *revert* that new change in case of potential
problems down the line (e.g., during deployment to QA).

Let's fill those functions with some concrete logic:

```python
def upgrade():
	# Add new columns 'first_name' and 'last_name' to the table 'contact'
	op.add_column('contact', sa.Column('first_name', sa.Text))
	op.add_column('contact', sa.Column('last_name', sa.Text))

	# Split 'full_name' and move into 'first_name' and 'last_name'
	results = op.execute("SELECT id, full_name FROM contact");
	for id, full_name in results:
		# Logic to split the name and insert into table
		first_name, last_name = full_name.split(' ')
		op.execute(f"UPDATE contact SET first_name = {first_name} WHERE id = {id}")
		op.execute(f"UPDATE contact SET last_name = {last_name} WHERE id = {id}")
	
	# Finally, drop the column
	op.drop_column('contact', 'full_name')

def downgrade():
	# Add 'full_name' column back.
	op.add_column('contact', sa.Column('full_name', sa.Text))

	# Join 'first_name' and 'last_name' into 'full_name'
	results = op.execute("SELECT id, first_name, last_name FROM contact");
	for id, first_name, last_name in results:
		# Logic to split the name and insert into table
		full_name = ' '.join(first_name, last_name)
		op.execute(f"UPDATE contact SET full_name = {full_name} WHERE id = {id}")
	
	# Finally, drop 'first_name' and 'last_name' columns
	op.drop_column('contact', 'first_name')
	op.drop_column('contact', 'last_name')
```

> The snippet above only serves as a simplified example to showcase a schema revision script. It is not an optimal logic. Be careful when actually splitting full name into first and last names. In some cultures, there are no last names, or the last names may consist of multiple space-separate words. Lastly, make sure to `RESTART` your identity columns. We don't want to accidentally cause an integer overflow in transaction ids. 

The `upgrade()` function above performs three important steps: (1) creates two columns,
(2) populates the two columns from an existing, older column, and (3) removes the older
column that is no longer needed. Not surprisingly, the `downgrade()` function is
the inverse operation of `upgrade()`: we add one column back, re-populate it from the two columns, and remove those two columns.

> It's best to keep migration scripts as small as possible. Even the snippet above could be split into multiple migration scripts. For example, in one revision, we can just add two columns. In the next one, we split the name into two parts and populate these two new columns. In the third and final revision, we get rid of the older column. Having smaller migration scripts allows users of the database to adjust their code without immediately causing breaking changes. More on [evolutionary database design](https://www.martinfowler.com/articles/evodb.html) later.

After creating the script, we can now run `alembic upgrade head` to apply the change in the database.

```bash
$ alembic upgrade head
INFO  [alembic.context] Context class PostgresqlContext.
INFO  [alembic.context] Will assume transactional DDL.
INFO  [alembic.context] Running upgrade None -> a1829f4e7900
```

In case something goes terribly wrong, we can also revert that revision:

```bash
$ alembic downgrade -1
INFO  [alembic.context] Context class PostgresqlContext.
INFO  [alembic.context] Will assume transactional DDL.
INFO  [alembic.context] Running downgrade a1829f4e7900 -> None
```

Interestingly (and actually quite importantly), Alembic runs the above revision script 
within a transaction. Transactions are atomic in SQL databases (e.g., PostgreSQL), so if
something fails within the transaction, it will be automatically rollbacked to the previous
state to keep the database consistent. That's amazing!

## Benefits of migration tools

With the help of tools such as Alembic, teams can seamlessly work together
on the schema changes and not be worried of corrupting the database state.
Such tools provide developers with a nice view of schema changes as scripts,
which makes it easier for them to keep track of, as well as, perform code reviews 
together. Migration tools are also a great addition to continuous integration and 
delivery (CI/CD) pipelines, which can run automated migrations in different 
environments such as Development and Production.

If you would like to learn more about applying Agile methodologies to databases,
check out the following articles:

- [Evolutionary Database Design by Martin Fowler](https://www.martinfowler.com/articles/evodb.html) describes a novel (at the published time) approach for database change management.
- If you are PostgreSQL user, their [documentation](https://www.postgresql.org/docs) is great along with the ["Don't Do This" best practices](https://wiki.postgresql.org/wiki/Don't_Do_This) wiki.

## Disadvantages

There is no silver bullet in software engineering. The same applies to using migration tools. Some of the biggest disadvantages are that:

- **Tools can be expensive.** Some of the tools come with great benefits ... at a $-value 
cost. I have personally not used such tools, but perhaps they can provide greater benefits
such as a separate UI for managing migrations.
- **It is unclear what schema looks like at a given point.** If schema evolves
rapidly, there could be a rapid flow of new revision scripts being created and applied to the database. These add more complexity and make it difficult for developers to determine what the schema looks like at a given point of time.
- **If something does not work, we need a new revision script**. If some migration script is found to have some logical issues (a.k.a. bugs) after it was already applied, another migration script is likely required to fix the issue, since modifying existing scripts could lead to corrupt database state.


## Final thoughts

Tools such as Alembic are great for managing change in database schema. As everything
else in life, they come with pros and cons that software development teams need to
consider before adapting such tools in their development lifecycle. In general, 
changing database schema can be a tricky thing and cause a nerve-wracking experience,
but it becomes much simpler and smoother with the right migration tools 
and processes (CI/CD) at hand.

## P.S. Wanna give Alembic a try?

I created a simple Dockerfile that will let you play with Alembic.
This assumes that you have [Docker](https://www.docker.com/) installed locally.

```Dockerfile
# Use an official PostgreSQL image as the base image
FROM postgres:latest

ENV PYENV="/app/alembic_env"

# Install necessary packages for Python and Alembic
RUN apt-get update && \
    apt-get install -y python3 python3-pip python3-venv && \
    mkdir -p "$PYENV" && \
    python3 -m venv "$PYENV" && \
    . $PYENV/bin/activate && \
    pip3 install alembic

# Set environment variables for PostgreSQL
ENV POSTGRES_USER myuser
ENV POSTGRES_PASSWORD mypassword
ENV POSTGRES_DB mydb

# Expose the PostgreSQL port
EXPOSE 5432
```

Copy and paste the contents above to your local computer. Then, build and run the image.

```bash
$ docker build -t postgres_alembic:latest . && \
	docker run -it postgres_alembic:latest /bin/bash
```