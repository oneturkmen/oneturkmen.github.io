---
layout: page
title: Research
permalink: /research/
---

**TL;DR**: My research is in the area of [*API misuse
detection*](https://sarahnadi.org/smr/api-misuse/), an intersection of program
analysis and software engineering.


**Longer description:** Instead of writing code from scratch, developers
frequently reuse existing functionality from other software (such as libraries,
frameworks). They reuse the functionality through so-called *Application
Programming Interface* (API), which is an interface between their own code
and someone else’s already implemented one. These APIs have numerous benefits such as:
- *Abstraction.* Developers do not need to know the fine implementation details
to reuse the existing code (i.e., just extend a class or call a function from the
library).
- *Easy integration*. Sometimes developers just need to add a few lines of code
and run a few commands to install a library as dependency.
- *Reuse*. No need to write that sorting algorithm, networking protocols,
assembly instructions, etc. from scratch!

Despite the numerous benefits, developers may incorrectly use such library and
framework APIs. Such incorrect usage, known as *API misuses*, can lead to bugs
that may not be detectable when you write, and only when you actually deploy
your application and it crashes at some point. My research is
therefore striving to help developers detect and prevent these bugs before
compiling the code.

Take a look at the following example of a classical misuse of Java Iterator API:

{% highlight java %}
public String getHeadName(ArrayList<String> names) {
    // Use iterator to print the names
    ListIterator<String> it = names.listIterator();

    // Bad!!!
    // Misuse! Do not call `next()` without calling `hasNext()`!
    String name = names.next(); // may throw NoSuchElementException

    // Good
    String name = "";
    if (names.hasNext()) {
        name = names.next();
    } else {
        // set default name
        ...
    }

    return name;
}
{% endhighlight %}


<!--**Other work:** Earlier in my master's degree, I spent some time in the area of
software variability and reuse (e.g., software product lines, variability
implementation strategies). We collaborated with the [IBM/Eclipse
OMR](https://github.com/eclipse/omr) team to help them better understand the
challenges and constraints in their software, and subsequently [provided
guidance](https://github.com/eclipse/omr/issues/5276#issue-630075221) on how to
tackle these problems.-->

