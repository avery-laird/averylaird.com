---
title: "Implementing the Piece Chain"
layout: post
categories: 
 - programming 
 - piece chain
 - the text editor
published: false
---

I wrote a [brief article](/programming/editor/2017/09/30/the-piece-table/) outlining my preliminary foray into the world of text editing data structures. I got a lot of awesome feedback, which I was totally unprepared for, but very encouraged by. Thanks to everyone who took the time to take a look. Whether you loved it, hated it, or somewhere in-between, I appreciate it!  

This article is going to focus mainly on how I plan to implement the piece table. More specifically, a look at some of the ways I *could* implement it, and a discussion about the merits of each method.  

Before we get started, there is something I should clear up. I want these articles to loosely flow from one to the other, and in the previous article, many people brought up a specific point. The first data structure I bring up -- and dismiss pretty quickly -- is a simple array. A few people pointed out that, while a plain old array isn't very efficient, there are some simple ways to improve its efficiency, usually by introducing other arrays. And that's totally true. But from my point of view, that's not an array anymore: it's another type of data structure. But also, it's literally not **an** array -- it's two or more arrays. I was talking only about a single array. Hopefully, that clears up some of the confusion.  

I will talk about the two major decisions I have to make: (1) language paradigm and (2) which data structure to represent the piece table.

# Pick a Paradigm

So we have three choices for language. I could use a procedural language like C, I could straddle the middle with C++, or I could use a fully object-oriented language like Python (or maybe Go). My second choice, how to represent the piece table, partially depends on my choice of language. For example, a doubly linked list would work very well in C or C++. I could implement something similar in Python. The question is: do I have a clear advantage to associating methods with textual elements?  

Let's refer back to [Charles Crowley's paper](https://www.cs.unm.edu/~crowley/papers/sds.pdf). He does a great job of laying out a basic structural hierarchy:  

### Domains / Types

`Item`
: a character
^
`Sequence`
: an ordered set of Items
^
`Position`
: an index identifying an Item, in the scope of a sequence (an integer from `0` -- `len(Sequence)-1`)

### Syntax

* `Empty`
* `Insert`
* `Delete`
* `ItemAt`  

He goes fairly more in-depth than that. But this gives us a rough idea of the things to consider.  


