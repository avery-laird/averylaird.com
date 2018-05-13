---
layout: post
title: "Implementing the Piece Table"
categories: programming
---

This was a lot harder that I thought it would be. First, I tried using a red-black tree to store the pieces, similar to the [method proposed](http://e98cuenc.free.fr/wordprocessor/piecetable.html) by Joaquin Cuenca Abela. He has already written a very impressive C++ implementation. The about red-black trees is that they are quick difficult to get right, and (in most cases) perform only slightly better than some alternatives. And, as it was once wisely said:

<blockquote>
  <p>
    &ldquo;...premature optimization is the root of all evil.&rdquo;
  </p>
  <footer><cite title="Donald Knuth">Donald Knuth</cite></footer>
</blockquote>


When thinking about balanced binary trees, there are always three main structures to consider: AVL trees, splay trees, and normal (unbalanced) BST trees. It may seem like a contradiction to include unbalanced BSTs in a list of balanced trees; however, UBSTs can remained balanced as long as the keys being inserted are sufficiently unsorted. A text editor doesn't usually meet this requirement.

AVL Trees usually perform very well, almost as well as red-black trees. If you're unsure about the nature of the data being stored in the tree, the AVL tree works nicely. We can work a bit smarter though -- there's a simpler and faster solution to consider: Splay trees. This structure redistributes the nodes of a tree similar to AVL trees. The main difference is that we do not aim to balance the tree at all -- we keep shifting around ("splaying") nodes until the most recently inserted node is the root of the tree.

At first glance, it might seem that splay trees should not perform well at all, perhaps even worse than the UBST, because it is not particularly balanced. Surprisingly, the amortized time complexity of operations on a splay tree is $$O(logN)$$. This is because splay trees rely on the assumption that recently accessed data is the most likely to be required soon in the future. This is generally true, but quite often true for text editors. Most of the time, we are working on only a small section of the document. 

Parts of Atom were [recently rewritten](http://blog.atom.io/2017/10/12/atoms-new-buffer-implementation.html) to use a splay tree, with apparently pretty good results. I know that splay trees will perform very well with the common use case -- I'm unsure how they will fare with large buffers and multiple cursors/concurrent editors. There's only one way to find out.

<blockquote>
  <p>
    &ldquo;One good test is worth a thousand expert opinions.&rdquo;
  </p>
  <footer><cite title="Wernher Von Braun">Wernher Von Braun</cite></footer>
</blockquote>

## Operations

When designing the buffer interface, two basic functions must be considered:

$$\begin{align}
\mathrm{insert}(index, string) \tag{1}\label{1} \\
\mathrm{delete}(span) \tag{2}\label{2}
\end{align}$$

A $$span$$ is an ordered set of two indices, $$(i_s, i_e)$$ which represents all characters within that range of indices (inclusive). In this post, I will handle only the insertion operation, and address deletion in a later post. 

The $$index$$ is known to us through the cursor position, and the $$string$$ is given by user input along with $$span$$. All other values must be handled internally. We are storing the information about edits in a piece table, abstractly; but actually, since we keep each piece in a tree, it's more of a "piece tree." There are some conditions that we must place on this tree, mainly that *an inorder traversal starting from the root of the tree results in a proper evaluation of the buffer*. Because a node is equivalent to a piece, and a tree is equivalent to a table, I will use these terms interchangeably (depending on what is most appropriate in the context).

Although we can properly evaluate the entire table by starting at the root, and any *section* of the buffer by traversing a child node and its subtrees, we do not know where the section belongs in the buffer. As I'll explain, there's no way to know a node's position in the buffer without any information about the rest of the tree. This is because we will use a very clever suggestion from Joaquin Abela, which allows us to store nodes based on their **relative index** --- their index relative to other pieces in the table. 

To implement this method, there are two changes we need to make to the typical splay tree: how we insert nodes, and how we store data. The procedure for inserting nodes is affected by the key we choose to sort the data. Intuitively, this should be something related to the position of a piece within the document. At first, we might consider using the desired insertion index, $$i_d$$.

The issue with using $$i_d$$ is that it must change for any insertion at index $$i \leq i_d$$. After such an insertion, we would have to update every node after $$i$$. In the case of several insertions at the beginning of an existing buffer, this cases a $$O(n)$$ time complexity for each insertion --- no better than an array.

Joaquin Abela suggests a [solution to this problem](http://e98cuenc.free.fr/wordprocessor/piecetable.html): store the offset information in the nodes themselves. He does this by storing subtree sizes, rather than indices: 

            Explicit Indexing                         Relative Indexing

                    n1                                       n1
                 index:  14                             size_right: 1
	             length: 10                             size_left: 14 
	            /         \           =>                length:    10
         n2                   n3                       /             \
	  index:   0           index: 23           n2                        n3
	  length: 14           length: 1      size_right: 0              size_right: 0
	 /          \         /         \     size_left:  0              size_left:  0
    A            B       C           D    length:    14              length:     1
	                                     /             \            /             \
                                        A               B          C               D

It's worth spending a bit of time talking about the example above, because there are two **very important** differences to consider. Firstly, note that `n2` has a index of `0` and a length of `14`, while `n1` picks up at index `14`. This is because size is 1-indexed (I do not consider strings of length `0` to have meaning) whereas index is 0-indexed. So, `n2` actually *spans* indices `0 -> 13`. I will also define an insertion at index $$i$$ to have the following behaviour:
    
	0123456789ABCDEFGH
    This is a sentence
	             ^       insert 'i' @ index D
    This is a senitence

Such that the index $$i$$ refers to the desired final position.

Secondly, and **most importantly**, note how the structure of the tree does not change in either case. This illustrates how the structure of the tree itself stores the correct order of the pieces relative to each other, and implies that any resources spent storing and updating the index is a waste. Unlike indices, this order is also constant even after splaying --- once the node is inserted properly, everything works fine.

Finally, we need to consider how an inserted piece may require the "split" of an existing piece. This involves devising a test to detect when a split should occur, and performing the split such that all properties of a piece table and a splay tree are preserved (I will not give a particularly in-depth explanation of splay trees here, but [the Wikipedia page](https://en.wikipedia.org/wiki/Splay_tree) is a good place to start).

### Insertions

To make an insertion, we will still need to consider the desired index, but we can forget it afterwards. I will go through an example below. I won't explain each splay tree operation, but they are easy to find online. We will use the same example as above, assuming the existing buffer was inserted as a single piece.

    (1) Original Tree        (2) Insert 'i' @ index 13

    size_right: 0            size_right: 0      Because:
	size_left:  0            size_left:  0      offset <index 13 < offset+length
	length:    18     -->    length:    18      we must split!
	text:     0x0            text:     0x0

                                  
    (2.1) View of subtree during split
	                /
	        size_right: 0
            size_left: 13
            length:     1
	        text:     0x1
	          /
    size_right: 0
	size_left:  0
	length:    13
	text:     0x2
	
	
	(2.2) Join split subtree with parent node
	
	                size_right: 0
	                size_left:  0   <-- must update size_left
	                length:    18   <-- must update length
                    text:     0x0   <-- must update text pointer
		              /
	        size_right: 0
            size_left: 13
            length:     1
	        text:     0x1
	          /
    size_right: 0
	size_left:  0
	length:    13
	text:     0x2
	
     
    (2.3) Update parent node
	
	                size_right: 0
	                size_left: 19  <-- 13+1+5
	                length:     5  <-- 18-13
                    text:     0x3  <-- explained below
		              /
	        size_right: 0
            size_left: 13
            length:     1
	        text:     0x1
	          /
    size_right: 0
	size_left:  0
	length:    13
	text:     0x2
	
Of course, there are actually 3 different ways to perform such a split. I choose to form a linked list on whatever side the node is supposed to be inserted, because we know there aren't any children there. The other thing to consider is how memory is managed throughout. If we examine just the memory addresses mentioned above, we might see something like this:

    (1)   0x0: This is a sentence
    
    (2.1) 0x0: This is a sentence
	      0x1: i
	
	(2.2) 0x0: This is a sentence
	      0x1: i
		  0x2: This is a sen
		  
	(2.3) 0x0: This is a sentence <-- here we can free(0x0)
	      0x1: i
          0x2: This is a sen
          0x3: tence

This method can be done without copying if we also store a `start` value in the node. We still need to allocate memory for the newly inserted character, but we would never have to change a pointer to memory once it is allocated. We can just increment the `start` value to be whatever the length of the first node in the split is. For example, the following memory contents and pointer/`start` pairs:

    0x0 This is a sentence
	0x1 i
    
	0x0, start = 0 , length = 13
	0x1, start = 1 , length = 1
    0x0, start = 13, length = 5
	
The `start` value would be relative to the text pointer. We can get the section of text we want by reading from `text+start` by `sizeof(text_type)*length` bytes.

Finally, if we do an inorder traversal of the example tree, we get:

    This is a senitence
    |-----------|||---|
	   piece1    |  |
	   piece2----|  |
	   piece3-------|
	   
An insertion without a split would form a subtree of a single node, and perform the same join operation. However, only the `size_left` of the parent node (in this case) would be changed --- the `length` and `start` values would not change. Actually, if we follow the process described above, a split will always join of the left side of a parent, because the desired insertion index is less than the span of that piece. 

## Code

First we will need a piece, which is just a collection of three values:

{% highlight c %}
typedef struct Piece {
    unsigned long start, length;
	char *text;
};
{% endhighlight %}

I'm assuming we're storing `char`s here, but we can replace this with a different type later if we have to. We will be storing these pieces in a tree structure:

{% highlight c %}
struct Tree {
    struct Piece *piece;
    struct Tree *left, *right, *parent;
    unsigned long size_left, size_right;
};
{% endhighlight %}

In terms of types, that's about it. I won't post a bunch of code here, as I will be putting it up on github. However, I will talk about the functions involved in insertions, and how I handle that process.

First, we follow the typical BST insertion algorithm. However, before performing the insertion, we perform a split test. If a split must be performed, we do it, otherwise, just insert the node normally. We record the address of the newly inserted node, and pass it to the splay function.

The splay function takes the address of the new node, and splays it up the tree until it is the root of the tree. That's it! Sounded pretty easy to me the first time I tried to write it. And, maybe if I had better coding chops, it would've been. However, after a while, I did manage to get a testable base implementation.

## Testing

It's hard to do useful, comprehensive tests in this case, since there are so many situations which may occur. However, I did some **very** rudimentary testing and benchmarking of the `insert` operation, with interesting results.

First, I inserted 1,000,000 characters into a piece table, which is equivalent to a dense document with 30,000 pages[^1]. giving the following graph:

![](/static/implement-piece-table/instantaneous-insert-time-1000000.png)

This is certainly a strange looking graph indeed, and that is mostly related to the nature of a splay tree. My guess is that the straight lines represent operations close to each other, and therefore very fast. The much slower insertion times represent operations very far from previous ones, which then become fast due to the splay operation. This is not bad for a worst case, but not particularly impressive either. Things get interesting if we take a look at a different quantity.

Inserting characters randomly doesn't give a fair representation of the common use case for most editors. To get a more balanced perspective, I also recorded the average time for each insertion into a table of a certain size. I kept a running total of insertion time, and after each insertion, I divided by the current table size. This reflects the average time for each insertion up to that point. I got the following result:

![](/static/implement-piece-table/average-1000000.png)

This suggests that, no matter the table size, it should always take about the same time to perform an insertion, and that is a result I'm happy with. As some simple, preliminary tests, there is not much that can be inferred --- perhaps a comparison between other implementations (array, linked list, etc) would be interesting.


[1^]: I'm using Joaqin's estimate here

<!--
### Insertions

There are a few important cases to consider testing during insertions: when the tree is empty, when a split occurs, and when a split does not occur. As mentioned before, we only have to test splits to the left, because a split to the right cannot occur (unless we change our sorting algorithm).

    (1) Empty tree  =>  assert: 
	 
	    NULL            new_node
		
	
	(2) Split, left child NULL  =>  assert:
	
	    node1                    last_half (node1->last_half)
		    \                      /    \
			node2            new_node   node2
                                /
	                      first_half
										  

    (4) No Split, left child NULL  => assert:
	
	    node1                       node1
		    \                      /    \
			node2            new_node   node2
    
    
	(5) No Split, right child NULL => assert:
	
	    node1                      node1
		/                         /    \
     node2                     node2  new_node
	 
 For each case, we'll compare the first tree and the result tree. Since `insert` modifies the tree in memory, we'll copy it first. What we want to example specifically in `first_half` and `last_half` is the `start`, `length`, and `size_<left/right>`.
 
#### Empty Tree: Insert 12 characters @ 0

| New Node |
|-|
|`size_left:  0`|
|`size_right: 0`|
|`length:    12`|
|`start:      0`|

#### Empty Tree: Insert 12 characters @ 1

Tree is still `NULL`! Print some errors about an attempted insert on out of range index.

#### Split: Insert 5 characters @ 1
	
| Old Node         | First Half     | New Node       | Last Half      |
|-|-|-|-|
|`size_left:  O`   |`size_left:  0` |`size_left:  1` |`size_left:  6` |
|`size_right: 5`   |`size_right: 0` |`size_right: 0` |`size_right: 5` |
|`length:     3`   |`length:     1` |`length:     5` |`length:     2` |
|`start:      0`   |`start:      0` |`start:      0` |`start:      1` |

Old Node is replaced with Last Half.
	 
#### No Split: Insert 25 characters @ 3
	
| Old Node         | First Half     | New Node       | Last Half      |
|-|-|-|-|
|`size_left:  O`   |`size_left:  0` |`size_left:  1` |`size_left:  6` |
|`size_right: 5`   |`size_right: 0` |`size_right: 0` |`size_right: 5` |
|`length:     3`   |`length:     1` |`length:     5` |`length:     2` |
|`start:      0`   |`start:      0` |`start:      0` |`start:      1` |

	
	
-------


In cases 1 & 2, we can perform a simple naive insert. Case 3 requires special attention during the *insert* stage, because we won't know the status of the buffer until we attempt the insert. Using the above example:

         n1
      index: 0
      length: 13
     /          \      => insert(10, <string of length 1>)
    1          NULL
		 
    normal BST insert:

         n1
      index: 0
      length: 13
     /         \ 
    1           n2
             index: 10 
             length: 1 
            /         \
           2           3

The parent node has a span `0<->(13 + 0)`, and the inserted node has a span `10<->(1 + 10)`. Therefore, before splaying, we need to modify the tree such that an inorder traversal will yield `0<->10, 10<->11, 11<->13`, rather than `0<->13, 10<->11` as it would now. We can do this by inserting a new node either **above** or **below** `n2`, and modifying the information stored in `n1` and `n2`. I will use the second method:

         n1
      index: 0          old_length = 13
      length: 9         length = n2->index - 1
     /         \ 
    1           n2
             index: 10 
             length: 1 
            /         \
           2           n3
                    index: 12    index = n2->index + n2->length + 1
                    length: 2    length = n1->old_length - n3->index + 1
                   /         \
                  3           4
				         
Note that `n3` will always be inserted as the right child of `n2`, because its index is the largest. The final step is to splay up the new node. Since we have created two new nodes, which one do we splay up? We should care only about the index in this case, because that is the data we're working on. Since we made an insertion at index 10, the final step is to splay up `n2`:

         n1                                                    n2
      index: 0                                              index: 10
      length: 10                                            length: 1
     /         \                                         /             \
    1           n2                                     n1              n3
             index: 10       => zig-zig right       index: 0        index: 11
             length: 1                              length: 10      length: 2
            /         \                            /         \     /         \
           2           n3                         1           2   3           4
                    index: 11
                    length: 2
                   /         \
                  3           4
						
An inorder traversal of the tree gives `0<->10, 10<->11, 11<->13` as desired. Note that we only need to introduce a second node if the node we just inserted has an index which intersects that of the parent node, which has three possibilities:

        |----P0----|
	|---P1--|   |--P2---|
	      |--P3--|
		  
Case 1 can only occur when `P1->index == P0->index`, and does not require a second node. Cases 2 and 3 both do require a second node, and are handled using the method outlined above. In fact, cases 2 and 3 can be considered the same case of making an insertion in an existing string. Therefore, the only test we need to perform is `node->index < (node->parent->index + node->parent->length)`. Note that an index equal to the parent nodes' offset is allowed, and is the same as case 1, expect that we insert the new node *after* the existing string and not *before*.

The cases above serve as useful illustrative examples, but we treat them as isolated from their subtrees. We cannot do this if the node has children: any index change in a node will also change the indices of its children, which we address in a [later section](#offset-propagation).

### Buffer Offset

Next we have to handle `start`: we can either maintain it throughout the tree, or keep a global counter in static memory which holds the offset. In the interest of steering away from global variables, we'll do it the first way.

Luckily, we are given `start` right away, as we must know the length of the string we are inserting. And, since the change buffer is append only, the value of `start` never changes once its set. The hard part is considering when we must "split" a piece --- when this happens, we need to find out the `start` of the new piece based off the old. Only one split can happen at once, so we are guaranteed to know the original `start` value from the node's ancestors. Consider the example below:

         n1
      index:   0
      length: 13
	  start:   0
     /         \ 
    1           n2
             index: 10 
             length: 1
			 start: 14
            /         \
           2           3

Here we have the insert sequence `insert(0, 13), insert(10, 1)`, before the tree has been fixed. We know the start value of `n2` is 14, since the previous node has `start = 0` and `length = 13`. We must perform a split to keep the tree consistent:

         n1                       n1
      index:   0               index:  0
      length: 13 (ol = 13)     length: 9
	  start:   0               start:  0 
     /         \         =>   /         \
    1           n2           1          n2
             index: 10               index: 10
             length: 1               length: 1      => splay n2
			 start: 14               start: 14
            /         \             /         \
           2           3           2          n3
                                           index: 12
                                           length: 4
                                           start: 10
                                          /         \
                                         3           4

The values of `n2` remain unchanged when we insert the new node, and we use the values from `n1` to calculate those for `n3`:

	n3->index  = n2->index + n2->length
	n3->length = ol - n1->length
	n3->start  = n1->length + 1
	
### Offset Propagation

The final step is to adjust the indices of all pieces after the point where a change occurs. When [splitting the pieces](#splitting-pieces), we didn't consider the children of the changing nodes, which is incorrect:

> Any time a change is made to the index of a node, the indices of its inorder successors must change as well
{:.aside}

This introduces a problem. In the worst case, we make many changes to the smallest node in the tree (the beginning of the buffer). Every other node is the inorder successor of that first node, forcing us to do a full traversal of the tree --- a $$O(N)$$ operation. Obviously, we will still have to traverse the full tree for certain things: evaluating the entire buffer, for example. We can refrain from this worst case by only updating indices when the full buffer is requested. But, if we allow partial evaluations, we run into another problem: we need to keep track of how much of the buffer has been evaluated, and at what time (relative to other edits). 

Joaquin Abela suggests a solution to this problem: store the offset information in the nodes themselves. He does this by storing the sum of the lengths of pieces inside a subtree. For example, a nodes's left subtree represents all of its inorder predecessors, therefore if we store the sum of the lengths of the pieces in a node's left subtree, that represents the offset for buffer until that node.


To give an example:

                    n1                                       n1
                 index:  40                             size_right: 1
	             length: 10                             size_left: 14 
	             start:  x                              length:    10
	            /         \           =>                start:      x
         n2                   n3                       /             \
	  index:  25           index: 51           n2                        n3
	  length: 14           length: 1      size_right: 0              size_right: 0
	  start:   x           start:  x      size_left:  0              size_left:  0
	 /          \         /         \     length:    14              length:     1
    1           2        3           4    start:      x              start:      x
	                                     /             \            /             \
                                        1               2          3               4

We replace `index` with two different fields, `size_left` and `size_right`. The benefit of this change may not be immediately clear --- basically, we have traded the absolute `index` for a "checkpoint" of the buffer state up until a given node. Note that calculating the `index` of any node is now of the same time complexity as calculating the buffer offset *used* to be. We have exchanged that information for something more useful to us.

How does this alter the insertion process? We certainly need to make some changes, because we were using `index` as the key to sort our data, and now it's gone. However, we still have an initial index which is given to us by the user. We use this index to figure out where the new piece initially belongs, because we can treat it as the `size_left` of the new node. I will show a quick example:

          
	  4   <-- length
	 0 6  <-- size left, size right
	    \
         1
        2 3
	   /   \
	  2     3
	 0 0   0 0

Above we have the pieces `(1, 2, 3, 4) -> (1, 2) -> (1) -> (1, 2, 3)`. Each set of parentheses represents a piece, and each decimal inside a piece represents the string's $$n^{th}$$ character, as related to its length --- we do not recognize strings of length `0`. Internally, this is stored in a zero-indexed array.  Note that `root->size + root->size_right + root->size_left = 10`, which is the total length of the buffer. You may notice that this tree appears to *no longer satisfy the requirements of a binary search tree*. This is because the key is no longer a primitive value stored in the tree. If we wanted to calculate a nodes actual key, we would obtain what used to be stored as its index. In other words, *the keys are not obtained until the tree is evaluated*. It's odd, but it works.

Suppose we would like to insert a string of length 2 at index 8 to the existing tree:

> an insertion is considered invalid if `index > root->size_left + root->size_right + root->length + 1`. We cannot make insertions at an index larger than the `EOF`
{:.aside.aside-sm}

To determine where the piece should go, we need to keep in mind a kind of "implicit key:" to insert piece in the tree above, we first compare the desired index with `0 + 4 = 4`. Since `8 > 4`, we move to the right subtree: `0 + 4 + 1 + 2 = 7 < 8`, so we once again move to the right subtree. Finally we have `0 + 4 + 1 + 2 + 0 + 3 = 10`, and we know the new piece must become the left child of the current node:

	  4   <-- length                 3
	 0 8  <-- size left, right      7 2
	    \             \            /   \
         1             3          4     2
        2 5           3 2        0 3   0 0
	   /   \   =>    /   \   =>     \
	  2     3       1     2          1
	 0 0   2 0     2 0   0 0        2 0
	      /       /                /
	     2       2                2
	    0 0     0 0              0 0
		         
                zig-zag-left     zig-right
	
If we evaluate the above tree, we get `(1, 2, 3, 4) -> (1, 2) -> (1) -> (1, 2, 3) -> (1, 2)`. There is a big problem, however: we *should* have split the second to last piece. How do we detect and perform splits using the data available to us? The answer is simple: we use the same technique as before, but with the index calculated at runtime, instead of the one that used to be stored in nodes. Below is the node which becomes the parent of the new piece:

    ... index = 7
      \ 
       3
      0 0

We check whether the desired index of `8` falls within the range `7<->7+3`. It does, so we know we must perform a split:

     ... index = 7
       \ 
        1 = ( 3 - (8-7) )
       0 4
          \ 
           2
          0 2
	         \          3                 1
	          2 = (total length) - (pre-span length)
	         0 0

We know that successive nodes must be the right children of the inserted node, because they must appear in order. We also know that this will **always** case the zig-zig-right case, so we could also perform the split by making the new node the root of the subtree right away. The final step is to splay up the newly inserted node:

	  4                      4                 2
	 0 8                    0 8               8 2
	    \                      \             /   \
         1                      2           4     2
        2 5                    4 2         0 4   0 0
	   /   \        =>        /   \    =>     \
	  2     1                1     2           1
	 0 0   0 4              3 0   0 0         3 0
	          \            /                 /
	           2          1                 1
	          0 2        2 0               2 0
                 \      /                 /
                  2    2                 2
                 0 0  0 0               0 0
                      zig-zig-right     zig-right
		
A very important characteristic is that the sum of sizes and lengths in the root must **never** change --- in other words: 

> once a piece is inserted or removed, no operation on the tree can change the buffer size.
{:.aside.aside-med}

If we traverse the new tree, we get the following sequence: `(1, 2, 3, 4) -> (1, 2) -> (1) -> (1) -> (1, 2) -> (1, 2) `. This is exactly as expected: by inserting at index 8, we split the last piece of the original sequence right on the second character, and we treat this as "before" the actual index. Most text editors behave this way.



## Test

I wrote a very simple splay tree implementation in C to test this concept. If I perform the insertions outlined above:

	insert(0, 13);
	insert(10, 1);
	insert(5, 3);

And traverse the tree inorder, I get the following output:

	index	start	length
	0		0		4
	5		16		3
	9		5		5
	10		14		1
	12		10		4

To put this example in context, it is equivalent to the following buffer operations:
 
    0 1 2 3 4 5 6 7 8 9 A B C D E F G
	T h i s   i s   a   t e s t          (1) insert(0, 13)
	
	T h i s   i s   a   t e s t          (2) insert(10, 1)
	                     ^                 
    T h i s   i s   a   t r e s t        (3) insert(5, 3)
	           ^
	T h i s   i f f f s   a   t r e s t
    |---------| |---| |-------| | |---|
	   0<->4    5<->8   9<->13 14 15<->17   




## Deletions




## Test



## A More Concrete Approach

My previous explanation and understanding were both very abstract. I realized that there are some key details missing:

**Implicit and Explicit Indexing**{:.sc}
: The order of the entries in the table matter. More specifically, we require the concept of a global, constant index to provided context for ordering the elements (the *true index*). We will refer to the order of the entries in a table as an *implicit indexing*, and the ordering of an entry relative to its position in the document as an *explicit indexing*. An important consequence is that we need to convert between these two orderings.

**Buffer Offset**{:.sc}
: To convert between implicit and explicit indices, we need to keep track of the position of each piece relative to the text. We'll call this the *buffer offset*. This is simply the sum of the lengths of all previous entries in the table. To avoid computing this value more often than necessary, we'll include it with each entry.

**Span**{:.sc}
: If a piece "spans" a true index, then the true index lies within the bounds of the piece, including the edges. It is not possible for a true index to lie between pieces.

Now we can flesh out the specifics in more detail. For example, what happens *exactly* during an insert?

1. Get the true index of the insert
2. Find the piece which already spans this index. If it exists:
   1. The true index lies strictly within the span, or on the bounds of the span.
   2. Within the span:
	  3. Truncate the piece 
3. If it does not exist, the insert must be at the EOF. Generate the entry, and insert it at the end of the table.

For example:

                    0  1  2  3  4  5  6  7  8  9  10
    Base buffer:    T  h  i  s     s  t  r  i  n  g
	Change buffer:     i  s     a

	buffer    start    length    (piece)
	base      0        4         1
	change    0        6         2
	base      5        6         3
	
For this example, the composite string has three pieces:

    This is a string
	|--||----||----|
     1   2     3
	 
If I wanted to insert some characters at true index 3 (corresponding to the vertical bar of piece 1), then I would make

    buffer    start    length
                                 T.h.i.s. .i.s. .a. .s.t.r.i.n.g.




[^1]: Here I mean that redundant or equivalent entries in the table will not be merged to reduce the size of the table-->


