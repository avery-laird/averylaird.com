---
title: "The Editor: Designing the Core API"
layout: post
categories:
  - programming
  - piece chain
  - the text editor
  - red black tree
published: false
---

It's been some time since I wrote my article regarding [data structures used in text editors.]({{ site.baseurl }}{% post_url 2017-09-30-the-piece-table %}) Since then, I have been thinking in more specific terms about how the core API could actually be implemented.  

## The Language

I have chosen C as my prototyping language, for now. I like C for this purpose because:
 
* It is **Clear and precise.** By writing in C, I get a lot of control over exactly how everything fits together. And although that may prove to be a burden in the long run, it's giving me some joy in the present.
* It **Plays Well with Others.** In the future, I would like to get extensions running nicely with the API. One amazing feature would be to allow extensions in other languages. This ability is already semi-latent in many modern languages today. Probably my biggest complaint with Emacs is the need to learn Elisp. I wouldn't complain so much if there existed great documentation, which there really doesn't, or if Elisp was a language worth learning, which it generally isn't (I realize that may be an unpopular opinion).

Additionally, C is not an object oriented language. This isn't so much a pro or con, but more an observation. I don't see a big advantage to using an object oriented language at this time, so I think this is fine. And if you're wondering why I won't just use C++ instead of C, it's because C++ is an awful language! If you don't believe me, ask [Ken Thompson](https://en.wikipedia.org/wiki/Ken_Thompson). If, in the future, the need arises for a shift in paradigm, I would consider Python or Go as likely candidates.  

## The Features

I got a lot of feedback from my last article, which was amazing. Two main suggestions popped out at me: *real-time collaboration* and *multiple cursors*. At this point, I need to decide whether these features should be incorporated as a part of the API, an Extension, or a Manager.

### Real-Time Collaboration

It's a good idea to consider this possibility early. And that's because working in real-time with others over the internet poses a whole [special set of challenges](https://en.wikipedia.org/wiki/Collaborative_real-time_editor#Technical_challenges) that need to be handled. Given my original definition of the API, which constricts the API functions to insertions, deletions, and reads, I think that collaboration should be implemented as an Extension. This may change in the future, as my concept of the API evolves.

### Multiple Cursors

I definitely want multiple cursors to be a part of the editor. First of all, it's important to consider the Manager. Namely, the Manager should have the ability to display *more than one cursor*, functionality which was not immediately obvious to me. This needs to be added to the requirements of a Manager. However, the mechanics of multiple cursors should be realized in the form of an Extension, largely for the same reason as above. 

## The Code

I've talked about the code for a while. Let's actually begin to take a look. Some obvious boilerplate is creating a type to keep track of our buffer:

{% highlight c %}
typedef enum BUFFER {
    ORIG,
    APPEND
} BUFFER;
{% endhighlight %}

Next, we need to set the groundwork for the Red-Black tree and the Piece Table. I would like to keep these separate, for two reasons:

1. To maintain the abstraction between a Piece in a Piece Table and a Node in a Tree
2. As a consequence of (1), keep the possibility open that Pieces may be stored in other ways (i.e. AVL tree, linked list, etc)

With this in mind, the Piece and Node are treated separately:

{% highlight c %}
/*!
 * This is a "Piece" in the most abstract, removed from the
 * tree functionality.
 */
struct Piece {
    BUFFER buffer;
    int start, length, size;
};

/*!
 * The rest of the information we need to actually store the 
 * piece in a tree is defined in the Node.
 */
struct Node {
    struct Piece *piece;
    struct Node *parent, *left, *right;
};
{% endhighlight %}

This is a slight trade-off of expansibility for space. I could have replaced the `struct Piece *piece` with the four values stored in a `Piece`. Then I wouldn't have to store an extra pointer in the `Node`, and I also wouldn't have to allocate the extra `Piece` in memory. The pointer itself is `8` bytes for a `64` bit machine, and the `Piece` is `16` bytes on my machine. Plus all the other pointers in the `Node`, it takes up `32` bytes in memory, so we have `32 + 16 = 48` bytes. If we take the stuff from `Piece` and put it in `Node`, that takes up `40` bytes. So we save those `8` bytes from the pointer, but that's completely negligible compared to the benefit we gain by separating the logic.  

Next, we define a `Tree` as pointing to a `Node`, and a `Table` as pointing to a `Tree`:  

{% highlight c %}
typedef struct Node *Tree;
typedef Tree *Table;
{% endhighlight %}

I'm still unsure about this. It might be better to define a `Table` as a structure, having storage. For now, this works. 


