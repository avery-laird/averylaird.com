---
title: "Text Editor: Data Structures"
layout: post
categories: programming editor
---

The first step in building my text editor is to implement the core API. If you're wondering why I want to do this, the original article is [here](/programming/2017/09/22/the-text-editor/).  

I researched several data types, and I tried to be *language agnostic*. I wanted my decision to not be influenced by any particular language, and first see if there was a "best way" out there, solely based on operations. Of course, a "best way" rarely exists. However, in the case of text manipulation and storage, there are some clear "worst ways" and "better ways." 

## The Worst Way

The worst way to store and manipulate text is to use an array. Firstly, the entire file must be loaded into the array first, which raises issues with time and memory. Even worse still, every insertion and deletion requires each element in the array to be moved. There are more downsides, but already this method is clearly not practical. The array can be dismissed as an option rather quickly.

> By the way: this isn't a challenge. Please don't try to find worse ways to manipulate text.

## A Good Way

Another option is a binary tree structure called a [rope](https://en.wikipedia.org/wiki/Rope_(data_structure)). Skip to the [next section](#a-rope) if binary trees aren't your thing.

> If you are unfamiliar with binary trees, check out [this](https://en.wikipedia.org/wiki/Binary_tree) as a starting point.

Basically, the string is split into sections, and stored in the leaves. The weight of each leaf is the length of the string segment. The weight of each non-leaf node is the total length of the strings on its left subtree. For example, in the diagram[^1] below, node $$E$$ is a leaf with a string segment $$6$$ characters long. Therefore, it has a weight of $$6$$, as well as its parent node. However, node $$B$$ has a weight of $$9$$, because nodes $$E$$ and $$F$$ together have a length of $$9$$.

![Source: Wikipedia, https://en.wikipedia.org/wiki/Rope_(data_structure)](/static/rope-data-structure.png){:style="display: block; margin-left: auto; margin-right: auto"}


This is a lot more efficient than an array. A rope has two main operations, `Split` and `Concat` (concatenation). `Split` splits one string into two strings at a given index, and `Concat` concatenates two strings into one. You can preform either an insert or delete with either of these two basic operations. To insert characters, you can split the string once (where you want to insert the content) and concatenate it twice (on either side of the inserted content). Deletions work similarly, by splitting the string twice, and concatenating them again without including the deleted content.  

There's a big downside. Using a rope is quite confusing and complicated. It's difficult to explain even in an abstract manner. Working out the kinks in real life, while still making the code maintainable and readable, seems like a nightmare. What's more, it still uses a lot of space. It didn't seem like the best option yet, so I kept looking.

## A Better Way

The [Gap Buffer](https://en.wikipedia.org/wiki/Gap_buffer) is much simpler than the rope. The idea is this: operations on text are often localized. Most of the time, we're not jumping all over the document. So, we create a "gap" between characters stored in an array. We keep track of how large the gap is by using pointers or array indices. Let's examine two cases (using pointers):  

**Insertion**
: The gap is a certain size to begin with. We copy over the inserted content, and if it exceeds the size of the gap, we expand the gap.
^
**Deletion**
: We shift the pointers of the gap to include the deleted content.  

This is makes a lot of sense. We are plagued somewhat by the same issues as an array; under certain circumstances, if we move too far from the gap, every element in the array will have to be moved. However, it's most likely that this is a rare occurrence for the average user. It is quite possible that the speed gained with most operations will outweigh the inefficiency of certain edge cases. In fact, the editor I'm writing this in -- [Emacs](https://www.gnu.org/software/emacs/) -- uses a gap buffer, and it's probably the fastest editor I've ever used. That fact alone is a pretty convincing argument to use a gap buffer. But if I'm starting from scratch, I want every aspect of the software to be the best option there is. And maybe there's a better(est) way.

## The Better(est) Way

A couple months ago, [my Dad](https://www.rosslaird.com/) asked me for help with a problem. He was converting one of [his books](https://www.rosslaird.com/stones-throw/) to markdown, and there was an issue with the footnotes. In markdown, footnotes do not automatically number themselves[^2]; they need to be labelled with either a number or some text, like this: `[^1]` or `[^footnote]`. The definition is the same, with a colon at the end.  

He had used pandoc to mostly convert the document, but every footnote had the format `[^#]`. It was my job to make a script to replace every `#` with a number, starting from $$1$$.  

Easy, right?  

Well, that's what I thought. I whipped up a regex, scanned through the document, and replaced all occurrences of the pattern with an increasing integer. And, it spat out garbage.  

Why? Because I had made a really, really obvious mistake. The counter doesn't always take up the same amount of space. The script kept overwriting content, and the offset grew larger the more footnotes were replaced. There's a simple fix: keep track of how much more space you take up, and add that to your current position in the document. I made that one simple change, and everything worked perfectly. Without knowing it at the time, I had used a [Piece Table](https://en.wikipedia.org/wiki/Piece_table).  

The Wikipedia page for the Piece Table is only 8 lines long (Yikes!) Even more concerning, it mentions Microsoft Word among the examples of editors that use piece tables. However, the piece table is a very promising structure. What's more, at its conception Word was lightning fast with infinite redo/undo, as explained in [this interesting article](https://web.archive.org/web/20160308183811/http://1017.songtrellisopml.com/whatsbeenwroughtusingpiecetables) by a Microsoft developer. If you have the time, it's a cool read.  

In 1998, Charles Crowley wrote [a paper](https://www.cs.unm.edu/~crowley/papers/sds.pdf) investigating the pros and cons of various data structures used in text editors. His paper includes the structures we covered, like gap buffers, arrays, and ropes. He concluded that -- from a basis of speed, simplicity, and structure -- the piece table was the leading method. From my point of view, the piece table is also the most elegant solution.  

We need two buffers: the original file (read-only), and a new file that will contain all of our added text (append-only). Lastly, we have a table that has three columns: file, start, length. This is which file to use (original or new), where the text segment starts in each file (pre-edit), and the length of the segment. Here's an example:  

    Original File: A_large_span_of_text   (underscores denote spaces)
	New File:      English_
	
	File           Start         Length
	-----------------------------------
	Original           0              2
	Original           8              8
	New                0              8
	Original          16              4
	
	Sequence: A_span_of_English_text
	
Keep in mind that the `Start` index is *not relative* to previous edits. This is something that gets handled at runtime by adding the length of each previous edit (like what I did to fix my footnote script). Since the length is already included in the table, this is a trivial step.  

I like this solution the most because:

1. It's **elegant**. Only the minimal amount of information is recorded. 
2. It's **simple**. We run through the table, and do the necessary operations. We don't need to re-balance or traverse a binary tree.
3. It's **fast**. This solution has the *potential* to be lightning fast. There are some additional optimizations to take into account, which are explained in detail [here](http://www.catch22.net/tuts/piece-chains)

The piece table method certainly has its complications, and there are different variations in implementation. It is certainly a daunting task. I'm going to see how far I get. Another article will accompany my attempts to implement the piece table method.

---

### Notes

[^1]: From [Wikipedia](https://en.wikipedia.org/wiki/Rope_(data_structure))
[^2]: They should! Why aren't they!?!?! Somebody needs to make that a markdown extension. Every time you want to insert an indexed footnote, you type `[^#]`. Then, it takes every footnote definition with that format and matches them up. If there's a mismatch (like a named reference), you wonder why some of your footnotes are missing and fix it. I had to change all of my footnotes just to insert this footnote. It's crazy. 
