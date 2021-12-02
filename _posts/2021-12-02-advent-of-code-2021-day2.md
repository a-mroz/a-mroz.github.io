---
layout: post
title: "Advent of Code 2021 - Day 2"
date: 2021-12-02 06:20:00 +0100
tags: [advent of code, competitive programming]
---

Day 2 is [behind me](https://github.com/a-mroz/adventofcode2021/blob/master/day2.py). I enjoyed it, even though it was still pretty easy (I believe that's going to change soon).


Let's take a look at the solution:


{% highlight python linenos %}
import fileinput


def parse():
    return [l.split() for l in fileinput.input()]


def task1(input):
    horizontal = 0
    depth = 0

    for cmd in input:
        dir = cmd[0]
        val = int(cmd[1])

        if dir == 'forward':
            horizontal += val
        if dir == 'down':
            depth += val
        if dir == 'up':
            depth -= val

    return horizontal * depth


def task2(input):
    horizontal = 0
    depth = 0
    aim = 0

    for cmd in input:
        dir = cmd[0]
        val = int(cmd[1])

        if dir == 'forward':
            horizontal += val
            depth += aim * val
        if dir == 'down':
            aim += val
        if dir == 'up':
            aim -= val

    return horizontal * depth


input = parse()

print(task1(input))
print(task2(input))

{% endhighlight %}


The task was similar to a one from the last year. In the input we have a `command value` tuples, hence the split in line #5. In the first part, we have a simple switch statement (and I just found out that it was recently added to python 3.10 in the `match-case` form. That's cool!). We simply modify two variables.

In the second part, we have the third variable, `aim` and a tiny change of logic. `down` and `up` modify the aim and `forward` modify *both* horizontal and depth variables.

That's it!

Ranks: 3562/2408. I guess it took me some time to wake up ;)