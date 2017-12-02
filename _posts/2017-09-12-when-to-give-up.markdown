---
layout:	post
title:	"The Consumer and The Dreamer"
categories: learning
published: false
---


I oscillate between two major schools of thought on how best to learn. The first school of thought can be called the Consumer. The Consumer approaches a topic very simply: by consuming and organizing all materials related to that topic.  

The second approach is that of the Dreamer. The Dreamer learns only by deriving a result independently, something Richard Feynman was famous -- or some would say *infamous* -- for.  

The Consumer and the Dreamer both have advantages and disadvantages. Although it may not seem immediately evident, I view these two approaches as opposite ends of the same spectrum. Like most things, neither extreme is the best approach. It is best to illustrate these differences through an example. The other day, I was wondering: is zero even, odd, or neither?  

## The Consumer

The Consumer may start like most people, and Google "the parity of zero." They would come across the first result, a [Wikipedia article](https://en.wikipedia.org/wiki/Parity_of_zero) entitled "Parity of zero." The information they need is the first sentence of the article:  

> Zero is an even number.  

But the Consumer does not stop there. They must read, or at least skim, the entire article. In addition, there are two important questions to ask:  

1. *Why* is zero even?
2. *Why* did I think zero was *not* even?

In other words, what is the deepest level of understanding I can get, and what caused my blind spot in the first place?  

In many cases, like this current example, there are many answers to the first question. Zero is even because:  

1. Parity alternates. $$-1$$ and $$1$$ are both odd, and zero sits right in the middle. Therefore, zero is even.
2. Zero is divisible by two (although to most people, this is not obvious).
3. A number $$n$$ is odd if there is an **integer** $$k$$ such that $$n = 2k + 1$$. If we say $$n = 0$$, then $$0 = 2k + 1$$ and $$k = -\frac{1}{2}$$. $$k$$ must be an **integer**; so we have a contradiction. Zero cannot be odd. (This might prompt further questions: if zero is *not odd*, then must it be even? Can a number be neither odd nor even?[^1])

There are many more reasons why zero is even, depending on how you think about it. I think the best explanation is the first: all numbers, *every single one*, follows this pattern. This explanation also brings forth -- naturally -- the answer to our second question. Why did I think zero was *not* even? Because I thought zero was special[^2]. There was a simple, obvious pattern the entire time, and I missed it completely! I fell victim to the common misconception that "zero is special," just like the third graders in [this](http://www.jstor.org/stable/30037740) 2008 paper[^3].  

The Consumer reads the entire Wikipedia article, including the "[Education](https://en.wikipedia.org/wiki/Parity_of_zero#Education)" section, which includes a figure from that paper (cited in the wiki) with claims made by the students.  

<!--
|--------------------------------------------------|
| Claims made by students                          |
|:------------------------------------------------:|
|*"Zero is not even or odd."*                      |
|*"Zero could be even."*                           |
|*"Zero is not odd."*                              |
|*"Zero has to be an even."*                       |
|*"Zero is not an even number."*                   |
|*"Zero is always going to be an even number."*    |
|*"Zero is not always going to be an even number."*|
|*"Zero is even."*                                 |
|*"Zero is special."*                              |
|--------------------------------------------------| 
-->

There is an interesting statistic in that section, which states that in a survey of nearly 400 seven-year-olds, $$45\%$$ thought that zero was even. What's more, in a second survey, when a third choice was added (i.e. *even*, *odd*, *don't know*), that number dropped to $$32\%$$. So, the majority of children (in other words, future adults) share this misconception.  

The Consumer methodology has several benefits. **Firstly**, the Consumer traverses a breadth of information. By expanding the context of the situation, the Consumer can draw inferences or make connections they might not have otherwise. This is a kind of "lateral learning" which fosters a generalized knowledge of things.  

<blockquote>
  <p>
  I don't know what's the matter with people: they don't learn by understanding; they learn by some other way — by rote or something. Their knowledge is so fragile!
  </p>
  <footer><cite title="Richard Feynman">Richard Feynman</cite></footer>
</blockquote>

**Secondly**, the Consumer finds the answer to their question almost immediately. However, that is just the end of the journey; it is essential to trace the path backwards, to get to the foundation of understanding. At first, the Consumer merely knows. Eventually, the Consumer understands.

Learning as the Consumer is not ideal. The previously mentioned benefits have significant downsides as well. Although "lateral learning" introduces you to other interesting topics -- or avenues of understanding -- it takes time to process the extra information, and skill to decide when to return to the main topic at hand. It is often difficult to know how "deep" to go. Furthermore, the Consumer method is "backwards;" by knowing the answer first, the Consumer may be less receptive to the foundation behind the fact. And that is the most essential.

## The Dreamer

The Dreamer begins with a question. Is zero even, or is it odd? Instead of looking up the answer like the Consumer, the Dreamer decides to figure it out independently. 

Now, if you're like me, you'd be pretty stuck at this point. Sure, we just saw a bunch of reasons why zero is even, 

<!--
The consumer does not read the article in a linear fashion. The Consumer functions much like a stack.  

## The Stack

The stack is an abstract data type, a very familiar concept to many with a computer science or engineering background. The idea has an almost perfect physical analogue: a stack (of things). Any physical stack of things will do; coins, laundry, books. The point is, you must deal with the thing on top before everything under it (ideally). If you have a stack of letters, you read and reply to them one by one, from top to bottom.  

I am not a fan of analogies, especially physical, to describe data types (or anything else). In this case it works fairly well, but quickly falls short; for example, in reality you can always read your letters or books out of order. Any analogy fails to convey the benefit that comes from what appears to be a restriction. The stack is perfect for situations in which a problem is broken down into an arbitrary number of other problems, which must be completed in a strict order. In other words, the solution depends on the result of each sub-problem (which, in turn, may have other sub-problems).  
-->

[^1]: Another interesting thing to consider is whether evenness/oddness is a binary property. In some ways, 4 is "more even" than 2, since 2 divides 4 twice but 2 only once. See [here](https://en.wikipedia.org/wiki/Parity_of_zero#2-adic_order). That means zero is the most even.
[^2]: As it turns out, zero *is* special in many ways, just not in this case. 
[^3]: Ball, Lewis and Thames. Making Mathematics Work in School.
