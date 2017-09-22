---
date: 2016-10-05 01:42:53+00:00
layout: post
categories: 
title: The Beauty of Sums
---
For many people, math is a scary thing --- something to be avoided. I think this is a shame, for many reasons, but one in particular always stands out for me. Pretty much anyone in Mathematics would recognize that one doesn't have to look hard at all to notice a subtle, profound and often breathtaking beauty in the realm of numbers.    

The fact that so many people often are denied this experience of wonder and awe resonates with me on a personal level. I used to *hate* math. In elementary school, math was taught to me in such a lobotomized and grey method that I never realized you could play with numbers (as with any other construct). My parents tried their best to keep me engaged, but it took many years to reverse the inertia built up by a host of teachers who also hated math. It wasn't until my mid-teens when I really began to nurture a love of numbers. And this mostly arose from meaningful play, which allows a certain familiarity and intuition to grow.   

There are so many great resources around today which show the profound connections in Mathematics, but I'm going to share one of my recent favourite discoveries, involving the sums of integers.

## The Beauty of Sums    

> proving the sum of all integers from $1$ to $t$, inclusive

I would like to know the sum of these numbers:    
$$
0 + 1 + 2 + 3 + \cdots + (t-1) + t
$$    
But, I don't want to count them every time. I want a simple equation which relates the sum of the numbers to $t$:    
$$
S(t) = 0 + 1 + 2 + 3 + \cdots + (t-1) + t = ?
$$    
First, I would like to construct a figure to visualize the sum. Let's picture each term in the sum as a rectangle, with width $1$ and height equal to the terms value. Next, align all of the rectangles next to each other (order doesn't matter, but either ascending or descending value is best) and look at the resulting rectangle:

<img style="display: block; margin-left: auto; margin-right: auto;" src="/static/media/uploads/natural-numbers-rectangle.svg" alt="" width="600"/>

Now, one of the first things you might notice is that the sum appears to be exactly half of the rectangle. But let's not assume, let's prove it. We already know that the rectangles we constructed are equivalent to the sum $0 + 1 + 2 + \cdots + t$. What is the "empty space" equal to?    

If we view the "empty space" for each individual rectangle, we see that the first term is $0$, the second is $1$, until eventually the empty rectangle is equal to $t$. Of course, we can simply write this as:    
$$
0 + 1 + 2 + \cdots + t
$$    
Know I hope it's clear that the "empty space" and our sum are actually equivalent. And together, they are equivalent to the total area of our constructed rectangle, which has sides $0 + 1 + 2 + \cdots + t$ and $t + 1$. Now we have a relation between all quantities of our figure:
$$
A = t \times (t + 1) = 2(0 + 1 + 2 + \cdots + t)
$$
And now we can simply solve for the sum:
$$
\frac{1}{2}t \times (t + 1) = 0 + 1 + 2 + \cdots + t
$$

There you have it! Now the real fun begins.

## Power's Out    
> proving the sum of the squares of all integers from $1$ to $t$

The first proof was very famous, and I think quite easy. There lies more beauty if we dig a little deeper. Now I would like to find a similar equation for this sum:    

$$
0^2 + 1^2 + 2^2 + 3^2 + \cdots + (t-1)^2 + t^2
$$    
First, let's construct a figure similar to the previous proof. To that, imagine constructing a square (not rectangle) for each term in the sum. Then, line them all up, side by side (again, order doesn't matter). As a result, a figure is created like the one below:

<img style="display: block; margin-left: auto; margin-right: auto;" src="/static/media/uploads/rectangle.svg" alt="" width="600"/>


The first thing to notice is that one side of the rectangle is the sum we just previously proved, $0 + 1 + 2 + \cdots + t$. So, right away we know that the total area of the rectangle is $\frac{1}{2}t(t + 1)$. The "empty space" is a bit more tricky to find than last time (it's definitely not just $\frac{1}{2}$) because both dimensions change with each term. Let's write out the empty space for some $t$, and see if a pattern emerges.    

From the figure, we know the empty space for $t=5$. I'll also write out the empty area (by term) for the first few $t$:    
$$
\begin{array}{c|c}
t & \text{empty space} \\
\hline
1 & 0 \\
2 & 1 \\
3 & 2, 2 \\
4 & 3, 4, 3 \\
5 & 4, 6, 6, 4 \\
6 & 5, 8, 9, 8, 5 \\
\end{array}
$$    
Have you found the pattern yet?

Of course, area is the product of length and width. The problem is that for each term, the length and width of the unused area changes. Here we'll use $t_0$ to represent the general term in  sum $S$. Let's find the empty space for the first few terms by using an empty space function, $E(t_0, t)$.
$$
E(1^2,t) = 1\times(t-1)
$$
$$
E(2^2,t) = 2\times(t-2)
$$
$$
E(3^2,t) = 3\times(t-3)
$$
Look at the right side of the equations above. If you split the products, do you notice anything special?

In general, we know that the length will be the term of $0 + 1 + 2 + \cdots + t$ at the equal index, and the width is the exact same sum but at a reverse index:
$$
\begin{array}{ccccc}
0 & 1 & 2 & 3 & 4 & \cdots & t \\
t & t-1 & t-2 & t-3 & t-4 & \cdots & 0 \\
\end{array}
$$
Keep in mind that this is not guesswork, or an assumption based on the pattern. I considered placing a more formal proof here, but it's really overkill. If we move from right to left, the general "empty area" square will always have height increasing by one for each term, and length decreasing by one. The product such that one quantity always increases by one, while the other decreases by one, is exactly what we have just described.

In row-column multiplication form, this would be represented as:
$$
        \begin{bmatrix}
        0 & 1 & 2 & 3 & \cdots & t \\
        \end{bmatrix}
        \begin{bmatrix}
        t \\ (t-1) \\ (t-2) \\ (t-3) \\ \vdots \\ 0 \\
        \end{bmatrix}
        = 0\times t + 1\times(t-1) + \cdots + t\times0
$$
Now we know (using our previous $E$ function) that the empty area for any given term -- in general -- can be described like this:
$$
E(t_0, t) = t_0\times(t-t_0)
$$
The final result is that the empty space in our special rectangle is actually the product of our first sum, in reverse.    

*What?*

Personally, I find that connection astounding. In the interest of stepping back to look at the big picture, let's review our general equation for the area of the rectangle:
$$
\begin{align}
\text{Area} & = (0 + 1^2 + 2^2 + 3^2 + \cdots + t^2) + \text{Empty Area} \\
& = \frac{1}{2}t(t+1)\times t = \frac{1}{2}t^2(t+1)
\end{align}
$$
$$
\begin{align}
\text{Empty Area} & = [0\times t] + [1\times(t-1)] + [2\times(t-2)] + \cdots + [t\times0] \\
& = \sum_{i=0}^{t}i(t-i)
\end{align}
$$

As far as crazy connections go, we haven't even gotten started yet.

> for a quick sanity check, you can express both the empty area and the sum of squares as sums from $i=0$ to $t$ and verify that they are equal to $\frac{1}{2}t^2(t+1)$

## The Mystery Triangle

A very interesting connection has been found between the sum of integers and this mysterious "empty space" quantity. The problem is that it doesn't really bring us any closer to finding a general equation for the sum of squares. We need to express the sum of the empty space in a different way. In order to do so, let's go back to our empty space function $E(t_0, t)$. Remember, this function takes any term $t_0$ in the sum which goes until $t$ (so it only returns the empty space for that given term).    

Given this function, I can describe the **total** empty space in terms of **individual** empty spaces, as opposed to one big sum. To illustrate, let's describe the empty spaces for $t=1$:
$$
E(0, 1) = 0\times(1-0) = 0 \\
E(1, 1) = 1\times(1-1) = 0
$$
The sum with $t=1$ is really $0^2 + 1^2$, so those two expressions are really me saying, "what's the empty space for $0^2$ when $t=1$?" And then, "what's the empty space for 1^2 when $t=1$?"

Another example when $t=3$:
$$
E(0, 3) = 0\times(3-0) = 0 \\
E(1, 3) = 1\times(3-1) = 2 \\
E(2, 3) = 2\times(3-2) = 2 \\
E(3, 3) = 3\times(3-3) = 0 \\
$$

As you'll notice, the $t$ parameter stays constant, while $t_0$ increments by one. To understand this better, I want to arrange each value of $E$ in a triangle, where each row represents $t$ and each element in the row represents a $t_0$, like this:

<img style="display: block; margin-left: auto; margin-right: auto;" src="/static/media/uploads/triangle-plain.svg" alt="" width="600"/>


In this case, the first row consists of only $E(1,2)$. The second and third rows are:
$$
\begin{array}
&E(1, 3)&E(2, 3)\\
E(1, 4)& E(2, 4)&E(3, 4) \\
\end{array}
$$
In terms of the triangle, the sum of each row corresponds to the total empty space for a sum, with one *very important* catch. The sum of the $(n-1)^{th}$ row of the triangle is the empty space for the sum of squares from $0$ to $n$. **The relationship is offset by one.**

Why?

Look at the peak of our triangle: $E$ starts off with $t$ already equal to $2$, not $1$. This is because otherwise, our triangle would have two rows with only one value (one of which being $0$) and that isn't very useful to us.

So we finally have another relationship: *the empty space of the sum of squares from $0$ to $t$ is equal to the sum of the terms in the $(t-1)^{\text{th}}$ row of our Mystery Triangle.*

## Recursion Submersion

We still have one more relationship to uncover before finishing our proof. It involves some more examination of the Mystery Triangle. The connection is illustrated by this figure:

<img style="display: block; margin-left: auto; margin-right: auto;" src="/static/media/uploads/triangle-with-sums.svg" alt="" width="600"/>


As you can see, each row of the Mystery Triangle can also be represented recursively by our first sum of integers! This is actually a sum of a sum, like this:
$$
\sum_{i=0}^{t}\sum_{a=0}^{i}a = \sum_{i=0}^{t}\frac{1}{2}i(i+1)
$$

Why does this connection exist? I have no idea. But we can use it to find our equation. First, we should write this equation in terms of the sum of squares:
$$
\sum_{i=0}^{n}\frac{1}{2}i(i+1) = \frac{1}{2}\sum_{i=0}^n (i^2 + i) = \frac{1}{2}\left[\sum_{i=0}^n i^2 + \sum_{i=0}^n i \right]\\
\text{Empty Area} = \frac{1}{4}i(i+1) + \frac{1}{2}\sum_{i=0}^n i^2
$$
Now we can use the above equation in our general expression for the total area, but with one very important modification. The above equation expresses the sum of the $t^{\text{th}}$ row of the Mystery Triangle, but we need the sum of the $(t-1)^{\text{th}}$ row.
$$
\text{Area} - \text{Empty Area} = \text{Sum of Squares} \\
$$
$$
\frac{1}{2}t^2(t+1) - \frac{1}{4}t(t-1) - \frac{1}{2}(0^2 + 1^2 + 2^2 + \cdots + (t-1)^2) = 0^2 + 1^2 + 2^2 + \cdots + t^2
$$
As you can see, $(t-1)$ as been substituted for $i$ in the empty area function.

Solving for the sum of squares is not hard, although extra care has to be taken when dealing with the two -- slightly similar, but distinct -- sums. One is from $0$ to $t$, while the other is from $0$ to $(t-1)$. To "merge" the two, keep in mind that the sums are identical until the very last term. It then follows that:
$$
\begin{align}
\frac{1}{2}t^2(t+1) - \frac{1}{4}t(t-1) & = (0^2 + 1^2 + 2^2 + \cdots + t^2) + \frac{1}{2}(0^2 + 1^2 + 2^2 + \cdots + (t-1)^2) \\
& = \frac{3}{2}(0^2 + 1^2 + \cdots + t^2) - \frac{1}{2}t^2 \\
\left[\frac{1}{2}t^2(t+1) - \frac{1}{4}t(t-1) + \frac{1}{2}t^2\right] \times\frac{2}{3} & = 0^2 + 1^2 + 2^2 + \cdots + t^2
\end{align}
$$
Although we could technically stop here, the expression can be simplified quite a bit more:
$$
\frac{1}{3}t^2(t+1) - \frac{1}{6}t(t-1) + \frac{1}{3}t^2 \\
= \frac{2t^2(t+1) - t(t-1) + 2t^2}{6} = \frac{2t^3 + 2t^2 - t^2 + t + 2t^2}{6}
$$
$$
= \frac{2t^3 + 3t^2 + t}{6} = \frac{t(2t^2 + 3t + 1)}{6} = \frac{1}{6}t(t+1)(2t+1)
$$