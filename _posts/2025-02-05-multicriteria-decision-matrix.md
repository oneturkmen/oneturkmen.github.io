---
layout: post
title: "Making Structured Decisions with MCDM"
date: 2025-02-05 09:31:12 -0700
description: Simple framework for making complex decisions.
tags: decision-theory
categories: decision-theory
published: true
---

Humans, like you and me, make decisions every day, however small or large. In
today's world with numerous options and factors, it becomes more important to
make decisions in a methodical way rather than relying on our "gut feeling". In
fact, some decisions we make require deep thinking due to potential
consequences and irreversibility ("you can't unring a bell" once you make
certain decisions). Beyond the two important factors, we may have limited
and dynamic contextual information that can easily overwhelm us.

To make structured, analytical decisions, we can leverage a very simple yet
powerful tool known as Multi-Criteria Decision Matrix, as described in the
book, ["Psychology of Intelligence
Analysis"](https://www.cia.gov/resources/csi/books-monographs/psychology-of-intelligence-analysis-2/),
by a former CIA veteran [Richards J. Heuer,
Jr.](https://en.wikipedia.org/wiki/Richards_Heuer). Before I introduce it,
let's understand the problem context to which it is most relevant.

## Problem: Buying a house is complicated 

Let us imagine the following scenario; let's say we would like to buy a house.
There are many factors involved in buying the house: obviously its price,
square footage, where it is located (e.g., close to or far from work?), the
crime levels in the surrounding neighborhood, quality of the building, future maintenance
fees, and so on.

We often buy the houses with our partners, and that can lead to conflicting
preferences. For example, what is more important: having a backyard, but paying
more, or perhaps vice versa? There is no "right" or "wrong" decision, just
different set of consequences (higher vs lower price; there is a backyard vs
there is none).

Buying a house, in general, is a high-stakes situation due to the amount of money
that we need to spend. We also often consider how long we are going to live at
that place, along with "investment" opportunities, as in how much it is going
to appreciate or depreciate over the next decade.

In short, it is a complicated and stressful process that takes weeks, if not
months. However, there is a way to simplify it by approaching the process from
the analytical mindset.

## Solution: Multi-criteria Decision Matrix

The essence of making a good decision is to gather and structure available
information at hand (the known knowns). Humans are not good at holding a lot of
information at the same time in their head. Thus, we need to *externalize* the
information onto a visible form, such as an A4 paper. In addition to the
externalization, we *break down* complex information into its simpler
constituents.

One way to externalize and break down the information in a structured way is
known as Multi-Criteria Decision Matrix (MCDM). It is a hands-on, paper-based 
method for making a decision that considers multiple constraints (criteria).
Let's see how it applies to the housing purchase situation above.

First, we come up with a list of attributes, such as:

- Location
- Price
- Square footage
- Estimated maintenance costs
- Whether there is a garden or not
- etc.

Next, we both assign relative importance to each of the attributes:

|**Attribute**|**% important**|
|-------------|-----|
| Location    | 30% |
| Price       | 25% |
| Sq Ft       | 20% |
| Maintenance | 15% |
| Garden      | 10% |

The ranked list of attributes helps focus on what matters most and leave out
things that do not matter. For example, is garden really important to us or can
we live without it? Are we okay with paying more but get a better location, or
vice versa, get a house further away but for a cheaper price? Should we include
*Maintenance* in the *Price* attribute rather than keeping it separate? Hence,
we can now make a more structured decision on what matters to us before
searching for a house. On top of that, we will immediately notice the
difference in our preferences and that of our partners, be able to quantify that
difference, and discuss it in detail.

Now that we have visited a few houses, we are ready to apply the structured
analytical method to make a decision. You now need to quantify each house's
"score": distribute (imaginary) 100 points among the options we have. For
example, in the table below, House A gets 50, House B gets 10 and House C gets
40 points for the same *Location* attribute. Simply put, we liked House A the
most, and House C is close enough to it, but we did not like House B as much;
all in terms of the "goodness" of location.

|                |                  |   | **Houses** |         |         |
|----------------|------------------|---|------------|---------|---------|
| **Attributes** | **% importance** |   | House A    | House B | House C |
| Location       | 30%              |   | 50         | 10      | 40      |
| Price          | 25%              |   | 20         | 50      | 30      |
| Sq Ft          | 20%              |   | 30         | 60      | 10      |
| Maintenance    | 15%              |   | 20         | 30      | 50      |
| Garden         | 10%              |   | 70         | 0       | 30      |


Having split 100 points across our options for each attribute, we can now
calculate the total score by multiplying the %-ge importance by the score of
the house and sum it up:


|                |                  |   | **Houses** |         |         |
|----------------|------------------|---|------------|---------|---------|
| **Attributes** | **% importance** |   | House A    | House B | House C |
| Location       | 30%              |   | 50         | 10      | 40      |
| Price          | 25%              |   | 20         | 50      | 30      |
| Sq Ft          | 20%              |   | 30         | 60      | 10      |
| Maintenance    | 15%              |   | 20         | 30      | 50      |
| Garden         | 10%              |   | 70         | 0       | 30      |
|----------------|------------------|---|------------|---------|---------|
| **Total Score**| **100%**         |   | **36**     | **32**  | **32**  |

<p></p>

Voila! In the table above, **House A** is the winner with total of **36**
points. House B and C have the same total scores despite scoring wildly
differently in some attributes, such as *Location* and *Price*. Note that the
points themselves only matter in *relative* terms, i.e., what house is better
than another and by how much.

We can also compare each of our results with our partners (if done
separately). The separate result will show the difference in both of our
preferences and help re-evaluate houses in case there is a vast difference
between our results.

You can extend the matrix and do *sensitivity* analysis to determine how much
change in one attribute's score can swing our decision from one house to
another. For example, we can calculate the amount that the House B's price has
to go down for us to make it our primary choice rather than House A. This kind
of add-on can help us understand how sensitive our choices are (firm on one
option vs being open to consideration of alternatives).

## Conclusion

In summary, we broke down a problem ("buying a house") into smaller
sub-problems (attributes & their relative importances for each option) and
externalized it onto a paper. These steps helped us evaluate and rank the options,
and ultimately make a decision.

Multi-Criteria Decision Matrix is one among many approaches to structured,
analytical decision making. It can be applied anywhere where the stakes are
high and a good enough decision has to be made, whether at home, in our
career, in relationships, or elsewhere. I highly recommend to read Richard Heuer's
book, as it is packed with valuable and insightful information on how to
effectively make sense of a vast amount of information in today's world.

## References

- Heuer, Richards J. Psychology of intelligence analysis. Center for the Study
  of Intelligence, 1999.
