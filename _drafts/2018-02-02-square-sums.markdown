---
layout: post
title: " Assembly"
excerpt: ""
categories: programming
---



Last summer, I took [Yale Patt](https://en.wikipedia.org/wiki/Yale_Patt)'s Introduction to Computing Systems course (or some variant of it). It was during the summer term at Zhejiang University, which is only a couple weeks long. That means we had around 6 hours of lectures each day, with labs due every few days. Even though the course itself was a terrific experience, the condensed format was challenging and necessitated some pragmatic prioritization of assignments (in other words, skipping the ones I couldn't do).

This isn't something that would normally bother me. However, one lab in particular has (sometimes literally) kept me up at night -- even after half a year. I'm not exactly why this is the case, but if I had to hazard a guess, it would be two reasons:

1. The lab was really *interesting*
2. I still don't fully understand the concepts of the lab

Reason #2 is a big problem for me. This is something I need to fix. Maybe when I do, the *ghost of labs unfinished* will let me get a full night's sleep.

## The Problem

This particular problem is designed such that a recursive solution is the easiest and most elegant. Often in computer science, examples of recursion include the [Fibonacci Numbers](https://en.wikipedia.org/wiki/Fibonacci_number), [factorials](https://en.wikipedia.org/wiki/Factorial), and so on. The problem is that these are terrible examples of recursion, *in computer science*. We would never use recursion to find a Fibonacci Number --- we would solve the recurrence relation. Factorials are [primitive recursive](https://en.wikipedia.org/wiki/Primitive_recursive_function), and so most compilers would replace your code with loops anyway. These all fail to answer the question of **why** recursion is so essential to computer science, because other, arguably superior, methods may be used. 

A better example of the power of recursion lies in solving a maze. Consider the maze below:[^1]

![maze](/static/maze/maze.svg){:style="display:block; margin-right:auto; margin-left:auto"}

We can divide the maze into individual cells, or blocks, like this:

![maze](/static/maze/maze-overlay.svg){:style="display:block; margin-right:auto; margin-left:auto"}

Consider some cell $$C_1$$. When at $$C_1$$, we may have up to four directions to choose from. If we choose a direction and move $$C_1 \rightarrow C_2$$, the same is true for our current position at $$C_2$$. Now imagine trying to design an iterative algorithm to solve the maze. A problem arises when any cell has more than one possible direction, because each subsequent direction choice relies on previous choices. An iterative solution is not *impossible*, however it is sufficiently complicated that we should consider an easier solution. 

One way to solve the maze is the following procedure:

{% highlight c %}
Path maze_solver( Path P, int row, int col) {
    if ( direction_is_valid( north ) ) {
        maze_solver( P, row - 1, col );
    }
    if ( direction_is_valid( south ) {
        maze_solver( P, row + 1, col );
    }
    if ( direction_is_valid( east ) {
        maze_solver( P, row, col + 1 );
    }
    if ( direction_is_valid( west ) {
        maze_solver( P, row, col - 1 );
    }
	
{% endhighlight %}

[^1]: From [mazegenerator.net](http://www.mazegenerator.net/)
