---
layout: post
title: "Becoming More Productive Software Engineer: Tips & Tricks"
date: 2024-02-24 12:05:00 -0700
description: Tips and tricks for getting more done in less time.
tags: swe productivity work
categories: tech
---

The year of 2024 is my 3rd year as a machine learning engineer at Borealis AI.
I previously spent 2 years as a software engineering researcher doing my
Master's degree at the University of Alberta where I collaborated with 2 teams
from IBM Canada. And even before my MSc, I spent couple summers working  at a
local startup in Bulgaria. Over time, I have garnered bunch of tools and
techniques that help my stay more productive as a software engineer.

## Automate it ... or maybe not

- What: Automate manual work.
- Why: Less time spent on mundane work and more time spent on things that matter more.
- How: Python scripting, bash is fine too although the more you write, the harder it is
to read and understand. Difficult to debug and reason about.
- When: whenever appropriate.

From 2019 through 2021, I had an opportunity to teach undergrad students at the University
of Alberta. I helped conduct programming labs for "Programming in C" and "Software Quality"
classes. The lab sessions of both classes led to program submissions made by students
which required manual grading. I would have to open each file, look through code,
then compile and test it. Sometimes, code won't compile. Once I even had a situation
where a few students zipped a bunch of files, but then *renamed* the file with
`.zip` extension to `.tar`. No, they did not convert it; they just changed the extension.

These files would take plenty of time to grade. Grading one submission would take me
at least 10 minutes because I would need to download the files, place them in the correct folder,
open code, read throught code and write comments
about it to the students. I saw opportunity in these manual and tedious tasks
because I had one of the most powerful tools a software engineer can have
- [Linux tools](http://www.cs.toronto.edu/~maclean/csc209/unixtools.html) wrapped in Python.
These seemingly simple tools helped save me lots of time and nerve, and let me focus
on providing the students with finer comments and feedback.

How did I do that? I mostly used these tools through Python scripts. The scripts internally
call underlying Linux tools (such as `grep`), parse their output and proceed
as needed based on what they see. For every manual step I did, I had a script for that.
If I could not automate the manual step, I would at least try to reduce context switching (e.g., switching
between code directories) and instead combine multiple code files into a single big file which I could
just scroll over.


```python
"""
Example of a script that loops over the current directory,
finds folders (e.g., one student's submission files),
and compile all C files inside. 
"""

import os
import glob
import subprocess

# Compiling files 
for dir in os.listdir('.'):

    # Rename folders to replace spaces by underscores
    if "." not in dir and " " in dir:
        new_dir = dir.replace(" ","_")
        os.rename(dir, new_dir)
        dir = new_dir
    # Get all C files
    f = glob.glob(dir + '/*.c')

    # Compile each C file with its own name using 'gcc'
    for c_files in f:
        print(c_files)
        subprocess.call(["gcc", c_files, "-o", c_files[:-2] + ".out"])
```

Overall, the scripts did not take long to write. It took me <30 minutes to write the scripts,
but they saved me hours of manual work.

> Automate repetitive manual work if you can. But make sure it is worth the cost. In other words,
there is no point in trying to automate something if the path to automation is going to take
more time than just doing the manual work directly.

## Ship early and often

- What: ship early and often
- Why: (1) no blockers; (2) more impact -> more visibility -> more career progression; (3) more flexible/agile.
- How: small PRs, even DEV deployments is good. Documentations, monitoring tools, configs, etc. Not just code.

During my work at Centroida back in 2018 and 2017, I did my best to ship as early as possible, as often as
possible. The company followed Agile/SCRUM process where we had a SCRUM master leading work defined in sprints,
usually one sprint taking 2 weeks. The startup was mostly in the survival mode most of the time (as startups
generally are), so we tried to move as quickly as possible and ship everything our clients asked for.

[Shipping early and often is not just helpful in
startups](https://www.ycombinator.com/blog/tips-ship-early-and-often/); the
advice is applicable everywhere, whether small or large company, industry or
academia. Shipping early and often let me get early buy-in from customers to
make sure I was building the right thing and not what I had in my head. For
example, when building an API, I wanted to first understand what the API needed
to do ultimately. After talking to product leads and other teams (e.g.
front-enders), I built a small API with a handful of endpoints that I thought
they needed. Note that the endpoints did not yet do anything complicated at
this point; they just returned some "test" data. I presented the API to my team
and was able to get early feedback and re-adjust accordingly. Now imagine if I
did not do that early enough. If I took days to build an API that *I thought*
my team needed, but in reality the expectations were completely different, I
would have wasted a lot of time and most likely got frustrated at having to
build a new API again.

When I get early buy-ins and deliver my work in small increments, it helps me build the credibility
of someone "who can get things done". It also shows that I avoid working in silos by asking 
immediate feedback early on and I would rather collaborate than do it all myself.

## Never be blocked

<Stopped here>


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

## Never be blocked

## Use checklists

- What: Use checklists. First use pen and paper to outline work to be done approx.
- Why: (1) get mental offload. (2) helps identify potential bottlenecks and gaps. (3) easier to communicate
first and get changes done instead of implementing it first and then integrating changes. (4) this is example
of design doc and reviews (TODO: cite).
- How: pen and paper, company docs/knowledge sharing platform, google docs. Ask questions - if I do this step by step,
will the work be completely done?
- When: medium or larger scope projects.
