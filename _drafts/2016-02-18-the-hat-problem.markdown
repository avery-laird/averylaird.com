---
date: 2016-02-18 22:35:51+00:00
layout: post
categories: 
title: The Hat Problem
---
There's a famous math problem of which I recently became aware, commonly called "The Hat Check Problem" (or some variant).
The problem usually goes something like this:
<p style="padding-left: 30px;width:95%">You are checking hats at a party, and neglect to mark the hats for later identification. As more and more people arrive, you begin to lose track of who owns which hat. When the party ends and people begin to leave, you hand out hats at random. What is the probability that everyone ends up with the wrong hat?</p>

There are a lot of predictions I made about what the probability (as a function of hats, or people) would look like -- all of them were wrong. The correct answer is awesomely inobvious.

Without including a huge explanation about <a href="https://en.wikipedia.org/wiki/Derangement">derangement</a>, I'll cheat a little and use the following series. For $n$ hats, the probability $P$ of **all incorrect matches** is defined as:
$$
P(n) = 1 - \frac{1}{1!} + \frac{1}{2!} - \frac{1}{3!} + \cdots + \frac{(-1)^n}{n!}
$$
It seems quite reasonable to say that as the number of hats -- $n$ -- increases, the probability of **all incorrect** matches would decrease. More hats, greater chance of a match, right?    

<blockquote>Here's an interesting side question: if the probability of <strong>all incorrect</strong> matches is some value $P$, is the probability of <strong>all correct</strong> matches equal to $1-P$?</blockquote>

Think about it:        
0. 0 guests, no hats, 100% probability of all incorrect matches    
1. 1 guest, 1 hat, 0% probability of all incorrect matches    
2. 2 guests, 2 hats, 50% probability of all incorrect matches    
3. 3 guests, 3 hats, 33.3...% probability of all incorrect matches


A general pattern is emerging, with two components:    
1. $P(n)$ has a tendency to alternate between a larger and smaller value.     
2. Each value decreases in difference from the previous value; eg, the larger values get smaller, the smaller values get larger.

So it would appear that $P(n)$ eventually converges to some finite value. Let's test our suspicions by applying the ratio test:
$$
\require{cancel}
L = \lim_{n \to \infty} \left| \frac{(-1)^{n+1}}{(n+1)!} \times \frac{n!}{(-1)^n}\right| = \lim_{n \to \infty} \left| \frac{\cancel{(-1)^n} \cdot (-1) \cdot \cancel{n!}}{(n+1) \cdot \cancel{n!} \cdot \cancel{(-1)^n}}\right|
$$
$$
= \lim_{n \to \infty} \left| \frac{-1}{n+1} \right| = \lim_{n \to \infty} \frac{1}{n+1}
$$
Since $L \to 0 < 1$ as $n \to \infty$, the ratio test says that the series is absolutely convergent and therefore convergent. Now it's clear that $P$ must approach some value, by definition of a convergent series.
<blockquote>The Alternating Series Test could also be used in this situation, since the terms are decreasing and $\lim_{n \to \infty} \frac{1}{n!} = 0$. However: keep in mind that the Alternating series test only proves that a series is convergent, <strong>not</strong> that it is divergent. Since the question was whether the series converges or diverges, the ratio test is a bit more robust.</blockquote>

One interesting thing to notice about this equation is the $n^\mathrm{th}$ power on the numerator. Notice that for all terms where $n$ is *odd*, that term will be *negative*, and vice versa. This is because a negative multiplied by a negative is a positive, but a positive multiplied by a negative is negative. To illustrate:
$$
(-1)^2 = (-1) \cdot (-1) = 1
$$
$$
(-1)^1 = -1
$$
Therefore, the sign of each term switches -- hence the name, **alternating series.**

By applying the ratio test above, I know that as $n\to \infty$ the probability of incorrectly matching all hats settles on a finite value. Now I want to know what that value is equal to.

###Estimating Alternating Series
Notice that the terms in the series switch between positive and negative, but their absolute values are always decreasing. That means the value of the series is "swinging" back and forth, but with a decreasing magnitude -- somewhat like a pendulum that eventually comes to rest. This series will eventually (or perhaps quickly) "come to rest" on its value.

Knowing this, we can begin to make some conclusions about the value of this series. Since the first term, $1$, and the second term, $-\frac{1}{1!}$ are equal, their sum is $0$ (and can be ignored). Therefore, $\frac{1}{2!}$ is the largest value in the series, $\frac{1}{3!}$ is the second largest, and so on. This means that the value of the series must lie somewhere between $\frac{1}{2!}$ and $\frac{1}{3!}$.

At this point, it may be becoming clear that we can keep getting better and better approximations by simply adding up more terms in the series. This intuitively makes sense. The question now is, **how can we measure the accuracy of our approximation?** And this question brings forth two additional, perhaps more tangible, questions:    

1. Is the error a quantity we can even measure, without actually knowing the true value of the sum of the series?     
2. If we can decrease the error of our approximation to an arbitrarily small value, can we prove the true value of the sum?

We have already placed bounds in the upper and lower limits of the sum. It can't be larger than $\frac{1}{2!}$, or smaller than $\frac{1}{2!} - \frac{1}{3!}$:
$$
\frac{1}{2!} - \frac{1}{3!} = \frac{1}{3} > \sum_{n=0}^\infty \frac{(-1)^n}{n!} > \frac{1}{2}
$$
<!---
In more general terms:
$$
\frac{1}{3} = P_2 - P_3 > P(n) > P_2 = \frac{1}{2}
$$
--->
This means the error is *up to* $\frac{1}{6}$, or $\frac{1}{2} - \frac{1}{3}$. So to answer our first question: there's no way to know the exact error in your approximation, unless you already know the value of the sum of the series. However, we can put an upper bound on the error. In this case, using the first two (nonzero) terms, the error is no more than $\frac{1}{6}$. It might be useful to know how the amount of error behaves as the number of terms increase, in order to decide how many terms to add in our approximation.

###Generalize
In order to be more rigorous, we need a better definition of error. That's easy: error is just **the amount by which one value differs from another.** In other words, the error is the difference between the estimated value and the "true" value. This is another way to explain why we couldn't figure out the exact error in the last section -- our definition of error says that we need **two** quantities to compare, but we only had one.    

Let the *partial* sum of the series $P$ from $0$ to $n$ be denoted by $P_n$, like so:
$$
P_n = \sum_{i=0}^n \frac{(-1)^i}{i!}
$$
$P_n$ is our estimation: in the previous section, we computed $P_3$ and found it to be $\frac{1}{3}$. Let $P$ be the infinite series, our "true" value, and $E_n$ be our error (using $n$ partial sum).  Using our more formal definition of error, we can write:
$$
E_n = \left| P - P_n \right|
$$
Why the absolute value? Depending on the approach used to get to this point, it shows up in the calculations. But more importantly, it doesn't make sense any other way. What does it mean to have negative error? Can you be more right than the right value? You're either wrong, by a positive amount, or you're completely correct (0 error). It's kind of like how cars don't have negative speedometer values. If you're going 60 km/h, you're going 60 km/h -- it doesn't matter what direction (*kind of* -- not exactly).     

Now we're ready for the real conceptual leap. **Get Ready!**    
Let's say I've computed some partial sum, $P_n$. Well, that's not the whole series, so every term after the $n^{\mathrm{th}}$ is ignored. But remember that every term *after* the $n^{\mathrm{th}}$ continues to decrease. So when I compute $P_n$ and skip the rest of the terms, how much of an "impact" could they have on the true value? At **most,** the term immediately *after* the $n^{\mathrm{th}}$, or the term with index $n+1$!     

Let $B(n) = \frac{1}{n!}$. This describes the absolute value of every term in $P$, with respect to $n$ (this is *not* an infinite series). Now, using our previous insight, we can append a crucial fact to our error equation:
$$
E_n = \left| P - P_n \right| \le B(n+1)
$$

As discovered above, the difference between the infinite series and the partial sum can be **as large as** the next term in the series (or in math symbols, $\le$). This means we could append some extra relationships to our error equation. Since we didn't know the value of $P$, we introduced a new quantity that we *did* know; the $n^\mathrm{th} + 1$ term.    

We did some rudimentary error calculations already, when summing the first 3 terms of the series, and found that the error could be at most $\frac{1}{6}$. Now we now know it's actually much smaller than that:
$$
B(3+1) = \frac{1}{4!} = \frac{1}{24} = 0.041\bar6
$$
If we graph $B(n)$, we can see how the maximum amount of error behaves as we increase the terms in our partial sum. It's not hard to guess how it will behave: $n!$ increases at a rapidly increasing rate, therefore $(n!)^{-1}$ decreases very quickly. Interestingly enough, a factorial is not defined for real numbers, only natural ones, which makes it a discrete function. Therefore, figuring out the specific rate at which it increases is not a trivial task.

There is still our second question left unanswered: if we can decrease the error of our approximation to an arbitrarily small value, can we prove the true value of the sum? We can't determine the exact error of our approximation, but we have placed an upper bound on it. At first glance, $B(x)$ decreases with a very rapid rate -- we can't easily take its derivative, due to the factorial, but it's clear that $\lim_{x \to \infty} B(x) = 0$. Of course, it wouldn't sense any other way: as we take a larger and larger partial sum, $P_n$ approaches the value of $P$, and the error of our approximation decreases.

So it appears we are stuck. We can get a decimal approximation of our infinite series, and estimate the error of that approximation, but there are two problems with that:   
 
1. How "close" is "close enough?" Without any context about the actual value of the series, the error isn't very useful. The "true" value could be tiny, so small that even a seemingly minuscule error has a large impact. Or, conversely, we could be adding up many more terms than we need to.    
2. Can we actually represent this value in a non-decimal form? Or are we doomed to always be at least a little impercise?

I have a solution that solves both those problems, by fighting fire with fire. That's right -- more infinite series!

###Taylor Series

If you're new to infinite series, $\sum_{n=0}^\infty \frac{(-1)^n}{n!}$ may not seem particularly significant. But actually, it's easily recognizable as a very common flavour -- especially if you've ever used Taylor Series.

The quick description of Taylor Series is that they are used to approximate functions (yes, more approximation!). The derivative of a function is used to represent it as an infinite sum at a point. In order to represent a function as a Taylor Series, you need:   
 
1. The $n^{\mathrm{th}}$ derivative    
2. The point at which to "center" the sum    

This is clear from the definition of a Taylor series:    
$$
\sum_{n=0}^\infty \frac{f^{(n)}(a)}{n!} (x-a)^n
$$
Let's take a really easy function to represent. Most are tricky to use in an **infinite** sum, due to the fact that their successive derivatives change. However, as you probably know, $e^x$ is a novel exception: it increases at a rate equal to itself. Always. That means we've already fulfilled the first requirement. For the second, we can just choose $0$.    

Now let's try constructing our Taylor series of $e^x$:    
$$
e^x = \sum_{n=o}^\infty \frac{e^0}{n!}(x)^n = \sum_{n=o}^\infty \frac{x^n}{n!}
$$     
At this point, it's starting to look pretty familiar. What's the difference between our Taylor series and original series? Instead of $x^n$ in the numerator, we had $(-1)^n$. Pretty close! What change can we make to our Taylor series, such that we will end up with our original series? We need to change $x$:    
$$
e^{-1} = \frac{1}{e} = \sum_{n=0}^\infty \frac{(-1)^n}{n!}
$$   
Perfect. Now we know that our infinite series is actually $\frac{1}{e}$.



####Hanging Up Our (collective) Hats

We started with a function, $P(n)$, which describes the probability of an incorrectly matched hat given $n$ hats. Then we created an infinite sum, $P$, which is $P(n)$ as $n\to\infty$:    
$$
P(n) = \sum_{i=0}^n \frac{(-1)^i}{i!}
$$
$$     
P = \sum_{i=0}^\infty \frac{(-1)^i}{i!} \approx \frac{1}{e}
$$
It's important to remember that the actual probability of an incorrectly matched hat *does* depend on the number of total hats; however, the $P(n)$ converges on a value so quickly that after about 4 or 5 hats, the probability is not affected in any large way. This is clear in the following graph, of the partial sums from $n=0$ to $6$:

<img src="http://averylaird.com/static/media/uploads/partialsum_n%3D0-6.png" style="width:100%">

This figure was generated using the following python code:    

    ::python
    import math

    def P(n):
        sum = 0
        for i in range(n):
            sum += math.pow(-1.0, i) / math.factorial(i)
            print "{0} hats --> {1}% probability of all incorrect matches".format(i, sum*100)
    P(10)

The `pyplot` code sections were removed for clarity, the implementation is trivial. As you'll notice, the output follows the same format I used in the very first section; you'll find that it matches up exactly.    

<a href="" id="how-is-it-useful"></a>
##What does this mean, and how is it useful?

We analyzed the problem. What knowledge did we gain by doing so?        

First of all, whether you have some experience with series and sums (and alternating series) or not, there was hopefully a new insight (or two) that you gained. But that's not all -- the great thing about the Hat Problem is that it's so **general.** Here's how I might rephrase the problem:

You have two ordered sets of things, $A$ and $B$. Both sets are of equal [cardinality](https://en.wikipedia.org/wiki/Cardinality), and in $\mathbb{R}$. For every element in set $A$, there exists a one-to-one relationship with an element in set $B$, stored in set $C$. A new set of one-to-one relationships, $D$, is created at random. Both $C$ and $D$ are in $\mathbb{R}^2$. What is the probability that **no**  vector in $C$ is equal to a vector in $D$?

Pretty abstract, right? But it's not hard to follow: one set $A$ or $B$ contains either people or hats, which of course need to be of equal size, and $C$ stores the information to **properly** match up the hats. In other words, $C$ stores the information the hat check *should have* been keeping, and the information every individual person knows (eg, which hat is mine again?) That means $A$ and $B$ need only store one piece of information (persons name, some form of "hat id", etc) but $C$ and $D$ need to store two pieces of information: each person, *and* which hat they own. That's why sets $A$ and $B$ belong to all real numbers, whereas sets $C$ and $D$ are in $\mathbb{R}^2$.

Next, we construct a fourth set $D$, which is very similar to $C$, but contains a randomized form of the information in $C$ -- analogous to handing out hats at random. The original problem asks, "what's the probability of *all* of the hats going to *all* the **wrong** people?" In our new definition, that's the same as asking, "what's the probability that *not a single* vector in $C$ contains the same information as a vector in $D$?"

<blockquote>Does it matter how many people and hats you have? What if set $A$ and $B$ are infinitely large?</blockquote>

Now the Hat Problem is in a form widely applicable to a huge variety of situations. **By adding a level of abstraction, we can see that the problem doesn't really apply just to hats, but many other things as well.** Any time you have two groups of things, and mix them up, the Hat Problem rears its head.

I hope you've enjoyed jumping down the rabbit hole with me. If you have any questions, comments, or find any mistakes, please email me at laird.avery@gmail.com or comment below.


<!--
For example, let's say there's a satellite in orbit, with which we'd like to communicate. It's very expensive, so we'd like to avoid sending the wrong instructions. However, right as we sent some crucial code, a solar flare jumbled up all the bits in our signal, and the satellite just received a random mix of information. What's the chance not a single instruction was correctly recieved? Pretty high! About $36.79\%$. We have set $A$, our instructions, and set $B$, our string of bits to accomplish the instructions. Of course, there are error detection protocols in place, and each bit is certainly not analogous to a single instruction. However, it gives us a rough idea of easily things can go wrong -- and how much engineering goes into simply preventing error.
-->







---


<!--For example, the error of an approximation using a sum of the first two terms cannot be greater than the difference of those terms:
$$
\frac{1}{2!} - \frac{1}{3!} = 0.3\bar3 \gt \sum_{n=2}^3 P(n)_{\mathrm{error}} \gt -0.3\bar3
$$

$$
\lim_{n\to \infty}(1 - \frac{(-1)^n}{n!})
$$
<p>It's probably the easiest to solve using the <a href="https://en.wikipedia.org/wiki/Squeeze_theorem">squeeze theorem</a>, but I bet I can do it in 6 lines of Python.</p>
<p>First, let's import the math module and name our function:</p>
<pre>#the_hat_problem.py<br />import math<br /><br />def P(n):<br />    # code here</pre>
<p>Next, we need to translate our general equation,&nbsp;$P(n) = 1 + \frac{(-1)^n}{n!}$, into Python code. First, we'll create a variable&nbsp;<code>terms</code>&nbsp;. For every hat, we will compute the term and add it to <code>terms</code>.&nbsp;Finally, we will subtract <code>terms</code> from 1.</p>
<pre>#the_hat_problem.py<br />import math<br /><br />def P(n):<br />&nbsp; &nbsp; terms = 0<br />&nbsp; &nbsp; for x in range(1, n+1):<br />&nbsp; &nbsp; &nbsp; &nbsp; terms -= (math.pow(-1.0, x)) / math.factorial(x)<br />&nbsp; &nbsp; return 1 - terms</pre>
<p>So, now we have a way to compute the probability of an incorrect match for $n$ hats. Let's test it out: for one hat, $P(1) = 0\%$, since there is only one possible match. For two hats, $P(2) = 50\%$.&nbsp;</p>
<blockquote>It's interesting to note that for the second two whole values of $n$, $2$ and $3$, $P(n)$ is perfectly approximated by $\frac{1}{n}$. For example, $P(2) = 0.5 = \frac{1}{n} = 0.5$</blockquote>
<p>Sanity check:</p>
<pre>&gt;&gt;&gt; from the_hat_problem import P<br />&gt;&gt;&gt; P(1)<br />0.0<br />&gt;&gt;&gt; P(2)<br />0.5<br />&gt;&gt;&gt; P(3)<br />0.33333333333333337</pre>
<p>&nbsp;So far, so good!</p>
<h2>This one goes to eleven</h2>
<p>Let's return to our original goal of determining the end behaviour of $P(n)$. Using a larger value of $n$:</p>
<pre>&gt;&gt;&gt; P(50)<br />0.3678794411714422<br />&gt;&gt;&gt; P(100)<br />0.3678794411714422</pre>
<p>That's weird... $P(50)$ and $P(100)$ are equal to at least 16 digits. It looks like the function converges on $0.3678794411714422$, but what is that numbers signifigance?</p>
<h2>If $\frac{d}{dx}i^x = i^x$, what is $i$?</h2>
<p>Chances are you don't recognize the number $0.3678794411\ldots$ at first glance. However, you might've heard of the number $e$. It just happens that&nbsp;$0.3678794411\ldots$ is the inverse of $e$, or $\frac{1}{e}$. At times like these, I find the quote below to be useful.</p>
<blockquote>In mathematics you don't understand things. You just get used to them. <br />- John von Neumann</blockquote>
<p>&nbsp;So how does $e$ relate to this equation? I really don't know, but at least we can tentatively say that: $$\lim_{n\to \infty}(1 - \frac{1}{1!} + \frac{1}{2!} - \frac{1}{3!} + \cdots + \frac{(-1)^n}{n!}) \approx \frac{1}{e} $$</p>
<p>We can prove, at least to 15 digits (the last one might be rounded) that this is true.&nbsp;</p>
<h2>What does it look like?</h2>
<p>I'm curious to see a graph of our data, let's try it:</p>
<pre>&gt;&gt;&gt; y = [P(x) for x in range(0, 100)]<br />&gt;&gt;&gt; plt.plot(range(0, 100), y)<br />&gt;&gt;&gt; plt.show()</pre>
<p>I get a nice graph that looks like this:</p>
<p><img src="/static/media/uploads/figure_1_hat.png" alt="" width="600" height="452" /></p>
<p>It's evident that $P(n)$ settles on $e$ quite quickly, by around $n = 5$ or $6$. That's really fast! By the time you have 5 hats in the mix, you're already looking at a probability of $\frac{1}{e}$ for zero matches. What about a higher resolution of $n$?</p>
<p>&nbsp;</p>
--->