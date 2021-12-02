---
layout: post
title: "Advent of Code 2021 - Day 1"
date: 2021-12-01 20:00:00 +0100
tags: [advent of code, competitive programming, python]
---

Here we go again. New Advent of Code has started today. I plan to take on the challenge and solve all the tasks before Christmas (even if some of them are solved later than the publishing day).

I don't care that much about my position on the leaderboard. I'm not a competitive programmer, and I don't stand a chance with someone who practices that, even semi-professionally. That said, I wouldn't mind getting at least one point ;)

Although I'm tempted to try some new programming language (recently Scala is on my radar) for now I decided to stick to Python. I'm not a Python programmer, so I'm pretty sure my code won't be idiomatic. Nevertheless, I like to tinker with this language from time to time. The event is a neat way to refresh some knowledge (I didn't have many chances to work with Python since the last AoC) and possibly learn new stuff.

Without further ado, let's take a look at [today's solution](https://github.com/a-mroz/adventofcode2021/blob/master/day1.py). The task can be found [here](https://adventofcode.com/2021/day/1), so I won't copy-paste it.

{% highlight python linenos %}
import fileinput

def parse():
return [int(l) for l in fileinput.input()]

def task1(input):
    previous_depth = None
    total_deeper = 0

    for depth in input:
        if previous_depth and depth > previous_depth:
            total_deeper += 1
        previous_depth = depth

    return total_deeper

def task2(input):
    previous_depth = None
    total_deeper = 0

    triplets = [input[i:i+3] for i in range(0, len(input))]
    for triplet in triplets:
        depth = sum(triplet)

        if previous_depth and depth > previous_depth:
            total_deeper += 1

        previous_depth = depth

    return total_deeper

input = parse()

print(task1(input))
print(task2(input))

{% endhighlight %}

Some explanations:

- Line #1 - `fileinput` is a neat [module](https://docs.python.org/3/library/fileinput.html) that I discovered last year. Makes reading from file or stdin easier.
- Line #4 - at first, here I had a loop with appending to the list. List comprehension looks so much better, though. It might be even slightly faster, but I would be careful with giving such opinions. And not that it matters here.
- Lines #6 - #15 - solution for her first task. Very imperative and probably not very pythonic. But does the work. In the loop we need to know the previous depth, so we use the `previous_depth` variable and compare against it in the loop. When the current value is grater than the `previous_depth`, we increment the counter. When the loop is done, we're done too.
- Line #21 - task 2 is very similar to the first one, but now we operate on a three-measurement sliding window. A quick and dirty solution, that I had in the first place, is to iterate over `range(len(input))` and sum `input[i] + input[i + 1] + input[i + 2]`. The caveat here is that we can get index out of range, so we need to protect against it. I didn't like this solution that much and changed it to the list comprehension. It looks much cleaner, but might be slightly harder to grasp by non-python developers.

Today's ranks: 2694/2855. I didn't read carefully and made an off-by-one error in the first part. Waiting for the whole minute is a pain, and I think it has significantly worsened my rank.

Let's see what tomorrow brings!
