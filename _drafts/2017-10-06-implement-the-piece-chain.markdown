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

He goes fairly more in-depth than that. But this gives us a rough idea of the problem's "shape."  

The type we need to focus on the most is `Sequence`. In the piece table method, a series of pieces will make up a sequence. We start with only one sequence, and generate more pieces as changes are made to the document. In practice, however, the [Manager](/programming/2017/09/22/the-text-editor/#the-editor-has-managers) would notify the [API](/programming/2017/09/22/the-text-editor/#the-editor-has-an-api) of the starting window size, and the API would start with a sequence around that size. This introduces some complications, namely pieces which span sequences. Also, it could be difficult to ascertain where sequences meet. Let's disregard those issues for now.  

In C, we might have something like this:  

``` c
typedef struct Piece {
    Piece * next;
    Piece * previous;
    
    // we might want to add a "logical offset" field later
    
    int length;
    int buffer;
} Piece;

typedef struct Sequence {
    Piece * first;
    Piece * last;
} Sequence;

```

In python and C++, it would be a bit different. Namely, sequences and pieces would both probably be classes. Moreover, the buffer might also be its own object, namely a [`memoryview`](https://docs.python.org/3/library/stdtypes.html#typememoryview) object (at least in python):  

``` python
class Sequence:
    def __init__(self, first, last):
        self.first = first;
        self.last = last;
	
    def insert(self, index, Buffer, length):
	    ...
		
	# other methods omitted
	
class Piece:
    def __init__(self, length, Buffer, back, front):
        self.length = length
        self.Buffer = Buffer
        self.back = back
        self.front = front
		
# memoryview omitted here

```



This assumes that we implement the piece table using a doubly linked list, and the details of doing so are explained very well in [this article](http://www.catch22.net/tuts/piece-chains). However, this is just one way of doing things. In the next section, I'll explore using a red/black binary tree instead.  

# The Testbed

In order to actually test the API, I need a Manager first. I didn't want to create one from scratch, before creating the API, so I decided to use [kilo](https://github.com/antirez/kilo) as a platform to preform testing. Kilo works well for my purposes, because it's small and simple. The editing functions are also separated and mostly independent, so replacing them should be relatively straightforward.  

There are a couple functions I'll need to replace:  

``` c
int editorOpen(char *filename);
void editorUpdateRow(erow *row);
void editorInsertRow(int at, char *s, size_t len);
void editorFreeRow(erow *row);
void editorDelRow(int at);
char *editorRowsToString(int *buflen);
void editorRowInsertChar(erow *row, int at, int c);
void editorRowAppendString(erow *row, char *s, size_t len);
void editorRowDelChar(erow *row, int at);
void editorInsertChar(int c);
void editorInsertNewline(void);
void editorDelChar();
```


This seems like a lot, but actually most of them are not particularly large. Kilo stores text in "rows," where every line is a row. Our piece table uses sequences, where pieces make up a sequence. The first function `editorOpen` reads a file, and converts lines to rows. Note that the whole file is read. We want to only read the viewport and a bit more. Also, we start with one sequence: a pointer to the start of the file
	



