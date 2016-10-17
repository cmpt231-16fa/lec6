<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DiKkJKvDi64-tree_road.jpg" -->
# CMPT231
## Lecture 6: ch13,18
### Red-Black Trees and B-Trees

---
## Devotional

---
## Outline for today
+ Red-Black trees
  + Insert
+ B-Trees
  + Motivation and concept
  + Search in \`O(t log\_t n)\`
  + Insert in \`O(t log\_t n)\`
  + Delete in \`O(t log\_t n)\`
  + Application to filesystems
+ Midterm review (lec1-5, ch1-12 x9)

---
## Balancing search trees
+ Complexity of most operations depends on **height**
  + Search, insert, delete
  + **Worst** case: tree becomes a **linked list**
+ One approach: regular **rotations**: new root for subtree
  + **Red-black** trees *(ch13)*
    + Levels alternate colour: *(max path) &le; 2x (min path)*
  + **AVL** trees: rotate after each insert/delete
  + **Splay** trees: on each search/insert/delete,
    + Rotate node to **root** and rebalance

---
## Red-black trees
+ BST tree, each node has **colour** *(1-bit)*
  + All leaves point to **sentinel** node: *nil* (black)
  + **Root** is black, parent is *nil*
+ Children of **red** nodes are always **black**
+ For each node, every **path** down to leaves has
  same number of **black** nodes (its *black height* )

![Fig 13.1, red-black tree](static/img/Fig-13-1.png)

---
## Height of red-black tree
+ Claim: node *x* of height *h(x)* &rArr; **black-height** *bh(x)* &ge; *h(x)/2*
  + Proof: every path to leaf has &le; *h(x)/2* **red** nodes
+ Claim: **subtree** at *x* has &ge; \`2^(bh(x))-1\` internal nodes
  + Proof: **induction** on height of *x*:
    + *x*'s **children** have black height &ge; *bh(x)-1*
    + by inductive hyp, their **subtrees** have &ge; \`2^(bh(x)-1)-1\` nodes
    + So num nodes in *x*'s **subtree** (incl. itself) is &ge;
      \`2(2^(bh(x)-1)-1)+1\` = \`2^(bh(x))-1\`
+ Claim: tree with *n* nodes has **height** &le; *2 lg(n+1)*
  + Proof: **Combine** above: \`n >= 2^(bh(r))-1 >= 2^(h/2)-1\`
+ Hence **tree operations** are *O(lg n)*

---
## Operations on red-black
+ **Read-only** operations are as in normal BST:
  + *search*, min/max, succ/pred
+ **Insert**ing: if *red*, is its **parent** red?
  + if *black*, does it change **black height** of ancestors?
+ **Delete**: no problem if *red*: won't **change** black height
  + if *black*: could affect *red* **parent**
  + could also affect **black height**
  + if deleting **root**, need new root to be *black*

---
## Tree rotation
<div class="imgbox"><div>
<ul>
<li> On a **node** in a BST
<li> **Preserve** BST property
<li> Colour-agnostic; applies to any **BST**
<li> **Right** rot similar
</ul>
![Fig 13.2, tree rotation](static/img/Fig-13-2.svg)
</div><div>
<pre><code data-trim>
def leftRotate( T, x ):
  y = x.right           # set y

  x.right = y.left      # reparent y.left subtree
  if y.left != NIL:
    y.left.parent = x

  y.parent = x.parent   # connect y to x.parent
  if x.parent == NIL:
    T.root = y
  else if x == x.parent.left:
    x.parent.left = y
  else:
    x.parent.right = y

  y.left = x            # reparent x to y
  x.parent = y
</code></pre>
</div></div>

---
## Rotation example

<div class="imgbox"><div style="flex:2">
![Fig 13.3, rotation](static/img/Fig-13-3.svg)
</div><div>
<pre><code data-trim>
def leftRotate( T, x ):
  y = x.right

  x.right = y.left
  if y.left != NIL:
    y.left.parent = x

  y.parent = x.parent
  if x.parent == NIL:
    T.root = y
  else if x == x.parent.left:
    x.parent.left = y
  else:
    x.parent.right = y

  y.left = x
  x.parent = y
</code></pre>
</div></div>

---
## Red-black insert
+ Same as regular **BST** insert: start with **search**
  + Insert new node where we **ought** to have found key
+ Colour new node *red* first
  + Then **fixup** to restore red-black property

---
## Outline

---
## Spinning-disk storage
<div class="imgbox"><div style="flex:3">
<ul>
<li> **Seek**: move head to **track**, <br/>
  wait for **sector** *(slow)*
<li> **Throughput**: read from <br/>
  consecutive sectors *(fast)*
</ul>
<div>
![Hard disk, CHS](static/img/Basic_disk_displaying_CHS.svg)
</div></div>

+ Lots of **small** *iops* (I/O ops/sec) are bad
  + So **buffer** and do I/O in larger *pages* at a time
  + Typical **page size** around *16KB*
+ **Seek times**: 15ms (laptop), 10ms (desktop), 4ms (server)
  + **Rotational** latency: 5.5ms (laptop), 3ms (server)
+ Typical **SSD** seek: 30ns
+ **Network FS**: much higher **latency**

---
## Trees for disk storage
+ To edit data on disk, **read** page into RAM, **modify**,
  and **write** page back to disk
  + **Limited** number of pages in RAM at a time
+ **Tree-based** disk filesystem: 1 *node* = 1 *page*
  + Want a **low**, bushy tree with large **degree**
  + Generalise **BST** to degree *t*

---
## B-trees
+ In a B-tree of **min-degree** *t*, every **node** *k* has:
  + \`n\_k\` **keys** in sorted order (*t-1* &lt; \`n\_k\` &lt; *2t-1*)
  + \`n\_k+1\` **child links**, interleaved between the keys
+ **Degree** of each node is between *t* and *2t*
+ All **leaves** are at same depth *h*
+ In terms of *t* and *h*, what is **min** num of keys stored?
  **Max**?
+ Variants: *B+*-tree: **payload** stored only in **leaves**
+ Variants: *B&lowast;*-tree: *2t-1* &lt; \`n\_k\` &lt; *3t-1*

---
## t=2 B-tree (2-3-4 tree)
(TODO: fig)

---
## Outline

---
## B-tree operations
+ **Search tree** interface: search, insert, delete
+ Assess not only **computational** complexity, but also
  + **Disk** accesses (read/write), in terms of *n* and *t*
+ Keep **root node** in RAM
  + Other nodes need to be **read** in from disk
  + Root may need to be **written** to disk if modified
+ Constraining **degree** (*t* .. *2t*) keeps tree **balanced**

---
## B-tree search

---
## B-tree insert

---
## Insert: example

---
## Outline

---
## B-tree delete

---
## Delete: example

---
## Delete example, cont.

---
## B-tree summary

---
## Using B-tree in filesystem

---
## Real-world filesystems

---
## Outline

---
## Midterm review

---
## Lecture 1: ch1-3

---
## Lecture 2: ch4-5

---
## Lecture 3: ch6-7

---
## Lecture 4: ch8,11

---
## Lecture 5: ch10,12

---
## Outline

---
<!-- .slide: class="empty" -->
