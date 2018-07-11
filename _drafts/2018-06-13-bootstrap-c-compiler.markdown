---
layout: post
title: "Bare-Handed Code: Bootstrapping C"
categories: c hardware assembly compiler
---

For the average programmer, the days of assembly (and machine code) are long gone. Depending on your IDE and typical language of choice, it might be more common to see machine code in memory. If you're like me, and spend most of your time cheerfully buoyant on the Dead Sea of abstraction, it's rare to dive deeper, and catch a glimpse of what lies beneath the surface.

![](/static/compiler/deadsea.jpg)

One interesting question to ask is, "how did we get here?" For example, consider the C compiler, which is written in C. Who's the chicken, and who's the egg? The answer is generally the same for all questions of this type: the egg. The important distinction to make is that the chicken which layed the egg and the chicken which hatched from the egg are not the same. Something *like* a chicken layed an egg, and out hatched the chicken we want. 

The C language is no different: there existed many iterations of C before we got the C we use today. And, like the mighty chicken, it continues to evolve (whether faster or slower than the chicken depends on who you ask). This kind of question got me thinking about the feasibility of making my own pilgrimage from assembly to C. I considered a process something like this:

* use **assembly** to make the simplest C compiler possible (**SCC**)
* use **SCC** to compile an extended subset of C
* rinse and repeat

What do I mean by "simplest C compiler possible?" In a loose sense, I mean the smallest subset of C functionality such that a compiler may be written, plus whatever functionality is useful or feasible for that end. The first step, writing a compiler in assembly, is easy to follow. The next step can be very confusing. This general process --- [bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(compilers)) --- is well-known in computer science. There are many different ways to approach the second step. To help clear up some ambiguities, it's best to be more clear about what SCC does exactly.

>I skip making my own assembler, because as long as I don't use any directives or pseudo-instructions, all assembly code can be trivially assembled by hand. Also, I'm using MIPS assembly, and most MIPS emulators do not accept machine code
{:.aside .aside-sm}

SCC takes an address in memory where the first character (code unit) of the source program is stored, outputs another memory address. The address points to valid assembly if the source is valid, or message to indicate the source is not valid if the source is not valid. In C terms, SCC takes a pointer to a array of code units (a string) and returns a pointer to another string: either assembly or an error (both in the same character encoding).

Say I wanted SCC to support simple arithmetic (ignoring, for now, the order of operations):

{% highlight c %}
i = 2 + 2;
{% endhighlight %}

We will assume that there is at least one free register in which to put `i`. In MIPS assembly, we could write:

{% highlight asm %}
addi $t2, $t2, 2 
addi $t1, $t2, 2    # result of 2 + 2 is stored in register t1
{% endhighlight %}

Now consider what kind of C program is required to go from one form to the other. A tokenizer is required, to split up the expression, and a parser, to generate a syntax tree. The original C compiler (one of them, anyway) used a [recursive descent parser](https://en.wikipedia.org/wiki/Recursive_descent_parser).

It is not immediately obvious (for me, anyway) how a compiler written in one version of a language can be used to compile a compiler for another version of the language. Of course, phrased in this form, the answer to this problem seems clearer. Say we have a compiled compiler $$C_A$$, which compiles language $$A$$, a Turing-complete language (ignoring memory limitations). $$C_A$$ may only compile language $$A$$. But if we want to create another language $$A'$$, we can write a compiler for $$A'$$ using $$A$$, and compile it using $$C_A$$. 

![](/static/compiler/compiler-flow.jpg)

If you've ever been to some sort of "team building retreat" for work or school, you've probably played a game that goes something like this: given a team of people, and a small number of "islands" (like a hula hoop or a foam pad, usually two or three), everyone must cross a certain distance without falling off the islands. The way to win is to keep a core group of people in the centre of an island, and slowly pass an island from the back to the front. Bootstrapping is very analogous to this game: $$C_A$$ is the current island, and we compile $$C_{A^{'}}$$ to "pass" an island to the front. 

So, the sticky nature of the second step has been solved. The next step in building a parser is defining a grammar. This makes the whole process a lot easier. I'll use Backus normal form (BNF) since it's one of the most commonly used grammar notations in computer science. I will not be completely formal in it's use however, so productions may contain a regular expression when convenient. The [entire syntax of C in BNF form](https://cs.wmich.edu/~gupta/teaching/cs4850/sumII06/The%20syntax%20of%20C%20in%20Backus-Naur%20form.htm) is not as long as you might expect (two or three pages). If we wanted to define a grammar which supports the example above, we could use something like this:

{% highlight text %}
<assignment-expression> ::= <unary-expression> <assignment-operator> <assignment-expression>

<assignment-operator> ::= =

<unary-expression> ::= <postfix-expression>

<postfix-expression> ::= <primary-expression>

<primary-expression> ::= <constant>

<constant> ::= [0-9]+
{% endhighlight text %}

There's more here than is actually needed just for that simple expression, but I adapted this from the BNF given in *The C programming language* to allow room to grow. 



