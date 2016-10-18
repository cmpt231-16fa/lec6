# Spare slide content here
## Not rendered in presentation

https://www.cs.usfca.edu/~galles/visualization/RedBlack.html

---
## Red-black trees
+ BST tree, each node has **colour** *(1-bit)*
  + All leaves point to **sentinel** node: *NIL* (black)
  + **Root** is black, parent is *NIL*
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

<div class="imgbox"><div>
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
+ Potential problems: new node is **root**, or
  + node's **parent** is already *red*
  + **Recolour** and/or **rotate** up tree to root

---
## Notation
<div class="imgbox"><div style="flex:2">
<ul>
<li> Let *z* be the newly **inserted** node (initially *red*)
<li> Let *p* be *z*'s **parent**
<li> Let *g* be *z*'s **grandparent**
<li> Let *y* be *z*'s **uncle**: i.e., **sibling** of *p*
</ul></ul>
</div><div>
![Fig 13-5a, notation](static/img/Fig-13-5a.png)
</div></div>

+ Only have problem when *p* is also **red**
+ We'll focus on when *p* is the **left** child of *g*
  + Hence *y* is the **right** child of *g*
+ Case when *p* is right child is **symmetric**

---
## RB insert: case 1
+ Case 1: *y* (uncle) is **red**
+ Then **recolour** *y* and *p* to black,
  + **Recolour** *g* to red, and
  + **Iterate**, with *g* as the new *z*

![Fig 13-5, RB insert case 1](static/img/Fig-13-5.svg)

---
## RB insert: case 2,3
+ If *y* is **black**, it must be *NIL*
  + No more **iterating** required
+ Case 2: *y* is **black**, *z* is a **right** child
  + Then **left rotate** around *p*: &rArr;
+ Case 3: *y* is **black**, *z* is a **left** child
  + Then **recolour** *p* black, and *g* red
  + **Right rotate** on *g*, and insert is **done**

![Fig 13-6, RB insert case 2/3](static/img/Fig-13-6.svg)

---
## RB insert: example

![Fig 13-4, RB insert](static/img/Fig-13-4.svg)

---
## RB insert: summary
+ Only case 1 (*red* uncle) **iterates** up the tree
  + No **rotations**, only **recolouring**
+ Once we have a *black* (NIL) uncle, we do either
  + **Two** rotations (case 2) and recolouring, or
  + **One** rotation (case 3) and recolouring
+ Final step: may need to ensure **root** is black
+ Complexity: *O(lg n)*, iterating up tree
