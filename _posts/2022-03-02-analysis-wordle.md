---
layout: post
title: "Deriving a good starting word for Wordle"
date: 2022-03-02 21:00:00 -0700
description: Basic analysis of 5-letter words. Inspired by Wordle.
tags: math probability wordle
categories: math tech
---


Back in December of 2021, I got enthusiastic about
[Wordle](https://www.nytimes.com/games/wordle/index.html), a game of guessing a
word. In a bit more detail, it is a game where you have 6 attemps to guess a
5-letter target word. What got me hooked into the game is probability. I wanted to
find answer to probability questions, such as "what would be the probability of
guessing a word from the 1st attempt?".

To answer the question above, we first need to get as many 5-letter words as
possible. In other words, we need to build a *population* of 5-letter words to
be able to calculate precise probabilities word occurrences, whether complete
(guessing entire target word) or partial (guessing one or more character of a
target word). While it may be difficult to get a full data set of all 5-letter
words in English, we can use some approximate datasets, such as the one with
5757 5-letter words from [the Stanford
GraphBase](https://www-cs-faculty.stanford.edu/~knuth/sgb.html). 

Probability of guessing a target word on the 1st try is `1 / (number of all
5-letter words)`. Evidently, it is a very low probability of success.
What we can look for instead is *good starting word(s)*. Those are the words
that will maximise our chance of guessing a target word in 6 attempts.

## Good starter words

We can use the following logic to derive a good starter word:

1. Calculate frequency of all letters (i.e., number of unique occurrences in words; do not double count). Sum all frequency counts.

```python
# Given a list of words,
"aaa"
"abb"
"abc"
"cba"

# we get the corresponding dictionary of frequencies:
{
    "a": 4,  # 'a' occurs in 4 words, and so on.
    "b": 3,
    "c": 2,
}
# and sum of all frequencies is:
total = 4 + 3 + 2 = 9
```

2. Calculate *weighted* letter "coverage", which shows the weighted percentage of letters (from alphabet of all words) covered by a word. The idea here is that we want to cover as many *unique* letters as possible for maximum diversity. The diversity helps us maximize likelihood of hitting at least one letter. We want a good starting word, not the "one-and-done" kind.

```python
"aaa"
# ==> take unique letter 'a'
# ==> freq['a'] / n
# ==> 4/9 = 44%
# Coverage: 44%

"abb"
# ==> take unique letters 'a', 'b'
# ==> (freq['a'] / n) + (freq['b'] / n)
# ==> (4/9) + (3/9) = 77%
# Coverage: 77%

"abc"
# ==> take unique letters 'a', 'b', 'c'
# ==> (freq['a'] / n) + ... + (freq['c'] / n)
# ==> (4/9) + (3/9) + (2/9) = 100%
# Coverage: 100%

"cba"
# ==> take unique letters 'a', 'b', 'c'
# ==> (freq['c'] / n) + ... + (freq['a'] / n)
# ==> (4/9) + (3/9) + (2/9) = 100%
# Coverage: 100%
```

3. Sort by coverage in descending order.
4. Drink some coffee and enjoy a chocolatine.

Hold on for a sec. `"abc"` and `"cba"` have the same coverages (of 100%). Which one is better? *Can* one be better than another?

##### Taking positions into consideration

In case of having two equally "good" words, we should consider proportions of each letter *in each position* (0th to 4th indices)
of each of the two words.

1. We look at each position "vertically" (over all *k*-letter words in a very large vocabulary) and 
calculate proportion (or "coverage") of each letter in a given position. 
For example, with the four 3-letter words above, we will have the following positioned coverages:

```python
# Position 1 ('aaac'):
{
    "a": "75%",
    # "b": "0%"  (not shown explicitly)
    "c": "25%"
}

# Position 2 ('abbb'):
{
    "a": "25%",
    "b": "75%",
}

# Position 3 ('abca'):
{
    "a": "50%",
    "b": "25%",
    "c": "25%"
}
```

We can also make an observation that the sum of relative frequencies at each position sums up to 100%. 

2. Given that *k* is a fixed number of letters/positions (it is `3` in this case), then for each word, for each letter, get the positional coverage, e.g.:

| Word        | Position 1  | Position 2  | Position 3  | Total | Total (relative = total / k) |
| ----------- | ----------- | ----------- | ----------- | ----- | ---------------------------- |
| "abc"       | 75%         | 75%         | 25%         | 175%  | 58.33%                       |
| "cba"       | 25%         | 75%         | 50%         | 150%  | 50%                          |

Voila. We found the best word of the two: `"abc"`. Congratulations, `"abc"`! :tada:

Actually, not quite. If you are diligent enough, you would see that `"abb"` has total (relative) coverage of 58.33%, i.e., the same as `"abc"`.
The problem stems from the repeating characters.

| Word        | Position 1  | Position 2  | Position 3  | Total | Total (relative = total / k) |
| ----------- | ----------- | ----------- | ----------- | ----- | ---------------------------- |
| "abc"       | 75%         | 75%         | 25%         | 175%  | 58.33%                       |
| "abb"       | 75%         | 75%         | 25%         | 175%  | 58.33%                       |

Why? Because, as letters start to repeat, the probability of hitting an existing letter from the target word becomes lower.
For example, `"abb"` has only two distinct letters, namely `"a"` and `"b"`, whereas `"abc"` has three. So, statistically speaking, `"abc"` has a higher
probability of hitting some or all letters correct.

You may wonder why then the two words `"abb"` and `"abc"` have the same relative coverage. The answer is that we should *not* be counting
positional coverage of a letter if we already saw it before. The following example shows the correct calculations:

| Word        | Position 1  | Position 2  | Position 3  | Total | Total (relative = total / k) |
| ----------- | ----------- | ----------- | ----------- | ----- | ---------------------------- |
| "abc"       | 75%         | 75%         | 25%         | 175%  | 58.33%                       |
| "abb"       | 75%         | 75%         | (skip)      | 150%  | 50%                          |

Notice how we skip adding the proportion of the letter `"b"` from position 3 because we already processed it in position 2.
This begs the question of what position should we choose if a letter occurs multiple times (i.e., in different positions). 
We just have to be *greedy*: for a given letter, we have to choose a position (and one only) that has the maximal proportion.
By choosing the maximal proportion, we maximize the end value of the total (relative) coverage.

Alternatively, we can just ignore the words with repeating characters before returning a set of good starter words
(ordered by total relative coverage). Note that we still use those words to calculate (relative) positional proportion (coverage)
of each letter.

Here is a list of top-10 starter words,
along with their total (relative) coverage (based on the dataset of ~5K 5-letter words):

```python
{
    'cares': 0.16803890915407332,
    'bares': 0.1677609866249783,
    'cores': 0.16737884314747262,
    'bores': 0.1671009206183776,
    'pares': 0.16616293208268193,
    'tares': 0.16581552892131318,
    'canes': 0.1657807886051763,
    'pores': 0.16550286607608128,
    'banes': 0.16550286607608128,
    'cones': 0.16512072259857566,
}
```

Aaand, we are done. Or so I hope.

## General Analysis 

Here I was just playing with letter frequencies. Check it out.

It's interesting that `"s"` is the most frequent letter
in positions 1 and 5 (0 and 4, resp., if you are a techie).

#### General letter frequency

Frequency of letters in all 5-letter words (counting duplicate letters within each word):

```
{
    'a': 2348, 'b': 715, 'c': 964, 'd': 1181,
    'e': 3009, 'f': 561, 'g': 679, 'h': 814,
    'i': 1592, 'j': 89,  'k': 596, 'l': 1586,
    'm': 843,  'n': 1285,'o': 1915,'p': 955,
    'q': 53,   'r': 1910,'s': 3033,'t': 1585,
    'u': 1089, 'v': 318, 'w': 505, 'x': 139,
    'y': 886,  'z': 135
}
```

#### Letter frequency at each position

```
{
    0: {
        'a': 296, 'b': 432, 'c': 440, 'd': 311,
        'e': 129, 'f': 318, 'g': 279, 'h': 239,
        'i': 74,  'j': 73,  'k': 91,  'l': 271,
        'm': 298, 'n': 118, 'o': 108, 'p': 386,
        'q': 39,  'r': 268, 's': 724, 't': 376,
        'u': 75,  'v': 109, 'w': 228, 'x': 4,
        'y': 47,  'z': 24
    },
    1: {
        'a': 930, 'b': 32,  'c': 82,  'd': 43,
        'e': 660, 'f': 12,  'g': 24,  'h': 271,
        'i': 673, 'j': 4,   'k': 29,  'l': 360,
        'm': 71,  'n': 168, 'o': 911, 'p': 113,
        'q': 10,  'r': 456, 's': 40,  't': 122,
        'u': 534, 'v': 27,  'w': 81,  'x': 33,
        'y': 65,  'z': 6
    },
    2: {
        'a': 605, 'b': 128, 'c': 184, 'd': 178,
        'e': 397, 'f': 87,  'g': 139, 'h': 39,
        'i': 516, 'j': 8,   'k': 90,  'l': 388,
        'm': 209, 'n': 410, 'o': 484, 'p': 169,
        'q': 4,   'r': 475, 's': 248, 't': 280,
        'u': 313, 'v': 121, 'w': 98,  'x': 67,
        'y': 68,  'z': 52
    },
    3: {
        'a': 339, 'b': 99,  'c': 210, 'd': 218,
        'e': 1228,'f': 100, 'g': 176, 'h': 73,
        'i': 284, 'j': 4,   'k': 243, 'l': 365,
        'm': 188, 'n': 386, 'o': 262, 'p': 196,
        'q': 0,   'r': 310, 's': 257, 't': 447,
        'u': 154, 'v': 61,  'w': 70,  'x': 5,
        'y': 41,  'z': 41
    },
    4: {
        'a': 178, 'b': 24,  'c': 48,  'd': 431,
        'e': 595, 'f': 44,  'g': 61,  'h': 192,
        'i': 45,  'j': 0,   'k': 143, 'l': 202,
        'm': 77,  'n': 203, 'o': 150, 'p': 91,
        'q': 0,   'r': 401, 's': 1764,'t': 360,
        'u': 13,  'v': 0,   'w': 28,  'x': 30,
        'y': 665, 'z': 12
    }
}
```
