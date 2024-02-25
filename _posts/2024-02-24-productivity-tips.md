---
layout: post
title: "Becoming More Productive Software Engineer: Tips & Tricks"
date: 2024-02-24 12:05:00 -0700
description: Tips and tricks for getting more done in less time.
tags: swe productivity work
categories: tech
---

The year of 2024 is going to be my 3rd year as a machine learning engineer at Borealis AI.
I previously spent 2 years as a software engineering researcher doing my Master's degree
at the University of Alberta, and worked for almost a year (although part-time) at a local
startup in Bulgaria. Over time, I have garnered a few techniques that help my stay more
productive as a software engineer.

## Automate it all (or maybe not)

- What: Automate manual work.
- Why: Less time spent on mundane work and more time spent on things that matter more.
- How: Python scripting, bash is fine too although the more you write, the harder it is
to read and understand. Difficult to debug and reason about.
- When: whenever appropriate.

Productive engineers dislike repetitive manual work. For example, if you have a large
CSV file where you need to change a column, it might take a while to manually open the
file, to correctly identify and change the column header and values. It's also error-prone since you
might accidentally delete the delimiter (e.g., comma) somewhere and may not be aware about
it until you (best case) or someone else (worst case) reads the file.

In such a case, just automate it. There is a plethora of libraries and tools
you can use to change the column. If you use Python, there is `pandas` and `polars` (among others).
If you prefer to do it using Unix commands, there is `awk`. E.g., with pandas it is as simply as:

```python
import pandas as pd
df = pd.read_csv("file.csv")
df["TARGET_COLUMN"] = 5 # change rows value of TARGET_COLUMN to 5
df.drop("TARGET_COLUMN", inplace=True)  # drop a column.
```

BUT! Remember, that not all manual work needs to be automated. If a process needs to be manually performed once,
and it takes less time to perform instead of writing automated scripts for doing that, then it's best to just
do it once. Sometimes, it's not only about time it takes, but also energy it takes to write a script because
in some cases automating a manual process may actually be complicated or even impossible (e.g., when
a human approval is needed).

## Ship early and often

- What: ship early and often
- Why: (1) no blockers; (2) more impact -> more visibility -> more career progression; (3) more flexible/agile.
- How: small PRs, even DEV deployments is good. Documentations, monitoring tools, configs, etc. Not just code.

Or don't get blocked. If your company follows Agile and Scrum processes, you most likely a sprint of 2 weeks each.
The goal here is to move quickly enough without compromising quality and safety. If you are blocked on something
(e.g., waiting for approval for production rollout), find something else to work on. In the worst case,
work on your professional development.

Another pitfall is the scope creep. For example, you are working on some new feature which is reusing some parts
of your code (e.g., utility library for date and time processing). You perhaps changed a function or two, but
then you notice that you can improve a few other functions inside that utility library. While you want to follow
the boy/girl-scout rule, you don't want to spend too much outside the scope of your task. If you really want
to optimize that module, just create a separate ticket (if you use Jira) to track that work and do it when
you have some free time or in the next sprint. In general, it's best to ask yourself - is it worth the effort?

## Use checklists

- What: Use checklists. First use pen and paper to outline work to be done approx.
- Why: (1) get mental offload. (2) helps identify potential bottlenecks and gaps. (3) easier to communicate
first and get changes done instead of implementing it first and then integrating changes. (4) this is example
of design doc and reviews (TODO: cite).
- How: pen and paper, company docs/knowledge sharing platform, google docs. Ask questions - if I do this step by step,
will the work be completely done?
- When: medium or larger scope projects.