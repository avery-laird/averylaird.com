---
date: 2015-09-23 08:17:41+00:00
layout: post
categories: 
title: Common Volume of n-Cubes in 3 Dimensions
---
<p>I was recently looking at some old ACM competition questions online, and stumbled across a couple&nbsp;terrific questions from the 2013 Berkeley Programming Contest. A particular problem caught my eye, and goes as follows:</p>
<p style="padding-left: 30px;"><strong>1.&nbsp;</strong>Write a program to compute the volume of the intersection of a set of cubes.&nbsp;The input will consist of several sets of cubes in free format. Each set begins with&nbsp;a positive integer, which indicates the number of cube descriptions that follow. Each&nbsp;description consists of four integers, x, y, z, and s, indicating the coordinates of a corner&nbsp;of the cube and the distance&nbsp;(s &gt; 0) that the three edges of the cube extend from that&nbsp;corner in the positive direction.&nbsp;For each set of cubes, output one line containing the volume of their intersection.</p>
<p style="padding-left: 30px;"><strong>Example:</strong></p>
<p style="padding-left: 30px;"><strong><img style="display: block; margin-left: auto; margin-right: auto;" src="/static/media/uploads/output.png" alt="" width="424" height="162" /></strong></p>
<p style="padding-left: 30px;">&nbsp;</p>
<p>You can get the full pdf <a href="http://www.cs.berkeley.edu/~hilfingr/programming-contest/f2013-contest.pdf">here</a>. From the minute I read it, this seemed like a pretty tricky question to solve.&nbsp;However,&nbsp;I thought doing it one dimension at a time might break it down into simpler steps. What evolved is a method I view to be pretty robust, and so far seems to work for an arbitrary number of cubes.&nbsp;</p>
<h2><br />One Dimension&nbsp;</h2>
<p>Take a set $C$ of two-dimenionsal vectors $\left\{ \begin{bmatrix}x\\\ m \end{bmatrix}, \ldots, \begin{bmatrix}x_n\\\ m_n \end{bmatrix} \right\}$ where $x$ is a coordinate on a number line, and $m$ is its&nbsp;<em>magnitude</em>, or length (described in the problem as "extending in the positive direction"). It can be proved that the length common to all vectors is the difference between the smallest sum of the magnitudes of the vectors in set $C$ and the corresponding component $x$, and the largest value of the vector component $x$. That's a mouthful!&nbsp;</p>
<p>Basically, I'm saying that if you have a set of lines, the shared length of those lines (assuming they <em>have</em> shared length) will be the smallest magnitude in the set plus its coordinate, minus the largest $x$ value in the set.&nbsp;</p>
<p>A column of vector sums $R$ can be created to easily find the sum of the vector components, $r$. The generated matrix will contain the sum of $x$ and $m$ for each vector in $C$:</p>
<p>$$R = \begin{bmatrix}&nbsp;x_1 + m_1 \\\&nbsp;x_2 + m_2 \\\ \vdots \\\ x_n + m_n \end{bmatrix} = \begin{bmatrix}&nbsp;r_1 \\\&nbsp;r_2 \\\ \vdots \\\ r_n \end{bmatrix}$$</p>
<p>The overlap of values in set $C$ can now be calculated by finding the smallest magnitude of all vectors in set $C$ and subtracting the largest value of $r$ in matrix $R$.&nbsp;</p>
<h3>Example</h3>
<p>Let's say we have 3 lines in set $L$, $l_1 = \begin{bmatrix}8\\\ 3 \end{bmatrix}, l_2 = \begin{bmatrix}9\\\ 5 \end{bmatrix},$ and $l_3 = \begin{bmatrix}0\\\ 10 \end{bmatrix}$. First we can generate $R$:</p>
<p>$$R = \begin{bmatrix}8 + 3 \\\ 9 + 5 \\\ 0 + 10 \end{bmatrix} = \begin{bmatrix}11 \\\ 14 \\\ 10 \end{bmatrix}$$</p>
<p>We take the smallest value in $R$, $10$, and subtract the largest $x$ component, $9$.</p>
<p>$$R_{min} - L_{max} =&nbsp;10 - 9 = 1$$</p>
<p>Therefore, the "common" length of these lines is $1$ unit.</p>
<h2>Two Dimensions</h2>
<p>Now we will use what we proved in the first section to find the overlapping area of squares in a two-dimensional space. We will use a new set $S$, for square. This is very similar to the previous example, however each vector in set $S$ contains three components instead of two, where the first two components are $(x, y)$ coordinates. I won't write down the definitions again, but let's do an example:</p>
<p>$$S = \left\{ \begin{bmatrix}0\\\ 0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}9\\\ 1\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}8\\\ 2\\\ 3 \end{bmatrix} \right\}$$</p>
<p>The vectors in this set can be split into two instances of the previous example and treated as two cases, meaning we can evaulate the common length <strong>once</strong> in the $x$ direction to obtain length, and&nbsp;<strong>once</strong> in the $y$ direction to obtain height. By "splitting" the vectors, we end up with two sets of one-dimensional vectors which I will call $X$ and $Y$:</p>
<p>$$X = \left\{ \begin{bmatrix}0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}9\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}8\\\ 3 \end{bmatrix} \right\}$$</p>
<p>$$Y = \left\{ \begin{bmatrix}0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}1\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}2\\\ 3 \end{bmatrix} \right\}$$</p>
<p>We can now perform the same operations as in the previous example, once on $X$ and once on $Y$.</p>
<p><strong>With respect to $x$:</strong></p>
<p>$$R_X = \begin{bmatrix}0 + 10\\\ 9 + 5\\\ 8 + 3 \end{bmatrix} =\begin{bmatrix}10\\\ 14\\\ 11 \end{bmatrix}$$</p>
<p>$$R_{minX} - X_{max} = 10 - 9 = 1$$</p>
<p><strong>With respect to $y$:</strong></p>
<p>$$R_Y = \begin{bmatrix}0 + 10\\\ 1 + 5\\\ 2 + 3 \end{bmatrix} = \begin{bmatrix}10\\\ 6\\\ 5 \end{bmatrix}$$</p>
<p>$$R_{minY} - Y_{max} = 5 - 2 = 3$$&nbsp;</p>
<p>Therefore, the shape formed by the intersection of the three squares has length $1$ and width $3$, and its area is $1 \times 3 = 3$.</p>
<h2>Three Dimensions</h2>
<p>We are very close to finding the common volume between $n$ cubes. At this point, we can find the common area of intersecting squares; now we just need to multiply that by the "common length" of components in the $z$ direction. Let's assume a set $C$ of cubes:</p>
<p>$$C =&nbsp;\left\{ \begin{bmatrix}0\\\ 0\\\ 0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}9\\\ 1\\\ 1\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}8\\\ 2\\\ 2\\\ 3 \end{bmatrix} \right\}$$</p>
<p>Next, we "split" the vectors:</p>
<p>$$X = \left\{ \begin{bmatrix}0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}9\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}8\\\ 3 \end{bmatrix} \right\}$$</p>
<p>$$Y = \left\{ \begin{bmatrix}0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}1\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}2\\\ 3 \end{bmatrix} \right\}$$</p>
<p>$$Z =\left\{ \begin{bmatrix}0\\\ 10 \end{bmatrix},&nbsp;\begin{bmatrix}1\\\ 5 \end{bmatrix},&nbsp;\begin{bmatrix}2\\\ 3 \end{bmatrix} \right\}$$</p>
<p>And finally, we find the length, width, and height with respect to the $x$, $y$, and $z$ directions:</p>
<p>$$R_X = \begin{bmatrix}0 + 10\\\ 9 + 5\\\ 8 + 3 \end{bmatrix} =\begin{bmatrix}10\\\ 14\\\ 11 \end{bmatrix}$$</p>
<p>$$R_{minX} - X_{max} = 10 - 9 = 1$$</p>
<p>$$R_Y = \begin{bmatrix}0 + 10\\\ 1 + 5\\\ 2 + 3 \end{bmatrix} = \begin{bmatrix}10\\\ 6\\\ 5 \end{bmatrix}$$</p>
<p>$$R_{minY} - Y_{max} = 5 - 2 = 3$$&nbsp;</p>
<p>$$R_Z = \begin{bmatrix}0 + 10\\\ 1 + 5\\\ 2 + 3 \end{bmatrix} = \begin{bmatrix}10\\\ 6\\\ 5 \end{bmatrix}$$</p>
<p>$$R_{minZ} - Z_{max} = 5 - 2 = 3$$</p>
<p>Therefore, the intersection of the cubes form a rectangular prism with length $1$, width $3$, and height $3$. The prism then has the volume $1 \times 3 \times 3 = 9$.&nbsp;</p>
<h2>The Code</h2>
<p>I have used this method to write some python code, which is able to calcuate the volume of an arbitrary number of intersecting cubes:</p>
<pre>class Cube:<br />    def __init__(self, x, y, z, s):<br />        self.coords = [x, y, z, s]<br /><br /><br />def calculateUnion(*cubes):<br />    """<br />    takes two cube objects, and calculates the volume<br />    of the shape formed by their union<br />    """<br />    magnitude_x = min([x.coords[3] + x.coords[0] for x in cubes])<br />    magnitude_y = min([x.coords[3] + x.coords[1] for x in cubes])<br />    magnitude_z = min([x.coords[3] + x.coords[2] for x in cubes])<br /><br />    length = magnitude_x - max([x.coords[0] for x in cubes])<br />    width = magnitude_y - max([x.coords[1] for x in cubes])<br />    height = magnitude_z - max([x.coords[2] for x in cubes])<br /><br />    return length*width*height</pre>
<p>&nbsp;And some examples of usage:</p>
<pre>&gt;&gt;&gt; c1 = Cube(0, 0, 0, 10)<br />&gt;&gt;&gt; c2 = Cube(9, 1, 1, 5)<br />&gt;&gt;&gt; c3 = Cube(8, 2, 2, 3)<br />&gt;&gt;&gt; calculateUnion(c1, c2, c3)<br />9</pre>
<p>&nbsp;As you can see, the code agrees with our own calculations.</p>
<h2>What next?</h2>
<p>This method is pretty general, and could be used for any number of calculations. Personally, I'd like to try higher dimensions next. Or, you could generate coordinates of cubes using a function $f(x)$ and see what happens. There are also some issues I have not addressed, for example, what happens if the cubes don't intersect? Or if two do, and one doesn't?&nbsp;</p>
<p>If you find any mistakes, have questions, or just have some comments to share, please feel free to either contact me at <a href="mailto:laird.avery@gmail.com,">laird.avery@gmail.com</a>,&nbsp;or comment below.&nbsp;</p>