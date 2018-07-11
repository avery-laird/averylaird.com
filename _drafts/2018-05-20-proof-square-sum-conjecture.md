---
layout: post
title: "Proof of the Square-Sum Conjecture"
categories: mathematics graph-theory square-sum
---

I came across a [great Numberphile video](https://www.youtube.com/watch?v=G1m7goLCJDY) where Matt Parker discusses his "Square-Sum" problem, which goes something like this: given integers 1 to $$n$$, can they be arranged such that each adjacent integer has a sum which is a perfect square?

In the video, he uses a graph to find exact solutions. This method treats each integer as a node, and two nodes are adjacent if (and only if) their sum is a perfect square. Then, a sequence which satisfies the "square-sum property" (SSP) can be found. In the process, this problem amounts to finding Hamiltonian paths in a graph, which are typically tricky problems to generalize.

Through the Hamiltonian path method, Matt found the following pattern: paths can be found for integers $$n=14$$ to $$n=17$$, $$n=23$$, and $$n=25$$. He conjectured that, for $$n=25$$ and up, a path can *always* be found.

To prove this conjecture, my best hope was to rephrase the question to not deal with Hamiltonian paths. Really, we don't care about finding any specific path, rather proving whether or not one may exist under certain conditions.

## Well, Colour Me Squared

Consider the graph below:

![General Graph](/static/square-sum/general-graph.jpg)

Each node corresponds to an integer in some sequence $$A = a_1, a_2, \ldots, a_n$$, where $$A$$ satisfies the SSP. The edges correspond to the perfect square formed by any pair of integers in the sequence:

$$a_i + a_{i+1} = e_i$$

Now let's say that every $$e$$ has a colour. This means that each perfect square within the range of the sequence has its own colour. Given a sequence of length $$n$$, we can figure out all possible perfect squares (and all possible colours). We want a function $$P(s)$$, which, given a perfect square and a length $$n$$, tells us how many unique ways to make that perfect square. For example:

$$
P(4, 4) = 1
$$

Because given the sequence $$B = 1, 2, 3, 4$$, there's only one unique way to make $$4$$: $$1 + 3$$ (unique means that we don't count $$3+1$$ separately). You might be thinking, "Wait! $$2+2=4$$!" which is, of course, correct. But remember, we only get to use $$2$$ once in the sequence, so this doesn't count either. $$P(s,n)$$ can be thought of as the number of times we may use a colour $$s$$ to colour the graph with $$n$$ vertices --- we'll call this quantity the colour's multiplicity.

<!--
$$P(s,n)$$ is not terribly complicated, because it follows a linear relationship. Given any perfect square $$s$$, we can make it by doing:

$$1 + (s-1), 2+(s-2), \ldots, (s-1) + 1$$

There are just two problems to fix: firstly, we are counting double the colours, because we are counting symmetrically (for example, $$1+3$$ and $$3+1$$). Secondly, if the square divides by two evenly, we need to subtract $$1$$, because we don't want to count colours of the type $$2+2$$ either. Perhaps the best way to handle this is to represent $$s$$ as a binary string, and use the following rules:

$$
P(s) = \left\{
  \begin{array}{lr}
    s \gg 2 & : s_0 \ne 0\\
    s \gg 2 - 1 & : s_0 = 0
  \end{array}
\right.
$$

Where the "$$\gg$$" operator performs a logical shift right. This corresponds to a division by 2. If the least significant bit of $$s=0$$, then we know that $$s$$ is divisible by 2, and so we also subtract 1 to avoid overcount. For example:

$$
P(4_{10}) = P(100_2) = 010_2 - 1 = 1
$$
-->

Now, a given colour is related to its multiplicity through $$P(s,n)$$, so we can keep track of it while colouring the graph.

<!--$$
E = \{\quad(i^2, P(i^2))\quad|\quad 2 \le i \le \lfloor \sqrt{2n-1} \rfloor\quad\}
$$

The notation will be $$e_1 = (2^2,P(2^2))$$. This gives us everything we need to start colouring the graph.-->

## Don't Be Squared

Let's start with $$n=13$$. We get the following uncoloured graph:

![uncoloured graph](/static/square-sum/uncoloured-13.jpg)

Next, we can try to colour the edges, while making sure that no vertex has two edges with the same colour:

![coloured graph](/static/square-sum/coloured-13.jpg)

You can see below that I kept count of how many times I had used each colour, and by the time I reached $$a_{12}$$, I ran out! Let's try adding $$14$$:

![coloured graph](/static/square-sum/coloured-14.jpg)

Note that the order I colour the edges doesn't matter, and it also tells me (basically) nothing about an actual solution. So, now that we have a method of determining which sequences work and which don't, let's generalize: for $$n \ge 25$$, can we always satisfy the SSP?

First let's nail down $$P(s,n$$:

$$
P(s,n) = 







