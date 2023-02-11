---
layout: post
title: "TODO: Something about managing database schema changes using git"
date: 2022-03-02 21:00:00 -0700
description: Managing database schema changes. 
tags: databases postgresql git
categories: tech
---


Today's world is dynamic and chaotic making it nearly impossible to predict
what is expected in future. Similar behavior can be observed in software development,
where requirements may drastically change within days even with careful, long-term planning.
That is perhaps why Agile methodology is a popular and perhaps the "standard" approach
to software development in today's world. <mark>TODO: Gotta improve intro. Focus on change management</mark>

Imagine a scenario in which you talk to a client about some cool mobile app
idea, and they ask you to store, process, and display some data in the app.
However, next day, they ask you for something else, so you need to address the
change in client's requirements on data. In consequence, you need to make
changes to the application and the way it stores, processes, and renders the state.

If we had something hard-coded in code that needed to be changed, it would be
simply changing the value of the associated structures (e.g., variables or
lists of strings). However, we typically use storage systems for storing the
"state" or data.

Among all storage systems, there are variety of types, such as relational and
non-relational (e.g., NoSQL) database systems. NoSQL databases, such as
MongoDB, have no fixed schema, so it is easier to adapt to ever-changing
requirements. In this post though, I would like to focus on changing the state in the
relational SQL databases, such as PostgreSQL, in scalable and effective ways.
In other words, how do we effectively manage changes in SQL databases knowing
that we have a strict schema to follow?


## Schema changes as migrations

Software requirements change frequently, and so are database schema associated
with them. Changing the schema is a tricky task that can easily result in data
inconsistency and even unexpected downtimes. Not surprisingly, making changes
to the schema can be a nerve-wracking experience.

The good news is that we are not alone. Software engineers have already
encountered hundreds of issues involving schema changes, which sometimes
resulted in lost billions <mark>ADD EXAMPLE HERE</mark> and even layoffs
<mark>ADD EXAMPLE HERE</mark>. We are here to learn their lessons.

If schema *changes* are important, then we should monitor them. We should
track who makes the change, when it was made, and why it was made. Such
approach is similar to *git* workflow, where incremental changes to code
are noted in history with their authors and "commit" (or change) reasons.

Akin to git "commits" that change application code, schema migrations
are changes to the source code that represent the schema. Those changes
can be applied incrementally (on top of each other) and can be easily reverted,
if something goes wrong.


- What are migrations
- Why are they needed.
- How to manage them effectively.
	- liquibase.
	- sqlalchemy with alembic.
	- migra.
- Benefits
	- Easy to review and nitpick.
	- Multiple people can work on the same database the same way they work on code (via git).
	- can revert if troublesome.
- Costs
	- Can have too many files, except migra.
	- If migration is found to be buggy, need another migration file.
	- not always clear what the schema looks like at a given point.

## Automating testing and deploying database changes with CI/CD 

- How can we automate migrations?
- How do we adapt Jenkins/GitHub actions to integrating database changes?
	- create a copy of a database, load dummy data
	- run tests
- How do we test database changes?
	- pytest
	- in-db tests (forgot how they are called)
- How do we deploy database changes to prod?
	- end-to-end tests.
	- may need to acquire exclusive locks on associated tables to migrate. But can do this in a scheduled way at night.
	- monitoring performance & alerts.

## Conclusion

- It is difficult. Surgeon's work bcz you don't want to lose data.
- Many considerations.
- Do backups, and do them often if you have an actively-used database.


Intro:
- Requirements change a lot, and so is state.
- State is usually stored in a storage system, such a database system.
- There are numerous well-known databases, such as PostgreSQL and MongoDB.
- While it's easy to adapt to changing schema in NoSQL, it is not straightforward in SQL.
- Popular SQL databases do not have flexible schema.
- On top of that, multiple people can make changes to a single database schema.
- How do we simplify the integration of multitude of changes introduced by multiple people?

Content:
- We treat schema changes the same way we treat source code changes.
- At a file level, we can track what's happening.
- But! The problem is that some changes may not be backward compatible.
- Imagine a scenario where we break down a single column into two.
- For example, "full name" becomes "first_name" and "last_name".
- How can we break it up without "surprising" the users of the database and not breaking any downstream code?
- We can make changes in stages.
- Stage 1: Bring new columns (we do not remove any existing ones).
- Stage 2: We relay information to the developers to adjust their code.
- Stage 3: We break the data from the old column to the new columns, and set up additional constraints if necessary.
- However, that's not great because we have to wait until consuming applications address changes in their code.
- In addition, a data race may occur when the old column gets modified while the new column stays the same, leading to inconsistency.
- Additional effort is needed to address such inconsistencies.

Conclusion:
- 

