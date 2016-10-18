<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-DiKkJKvDi64-tree_road.jpg" -->
# CMPT231
## Lecture 6: ch18
### B-Trees and Midterm Review

---
## 2 Corinthians 5:17-19 <span class="ref">(NASB)</span>
Therefore if anyone is **in Christ**, he is a **new creature**; <br/>
the **old** things passed away; behold, **new** things have come.

Now all these things are **from God**, <br/>
who **reconciled** us to Himself through Christ <br/>
and gave us the **ministry of reconciliation**,

namely, that God was in **Christ** <br/>
**reconciling** the world to Himself, <br/>
not counting their **trespasses** against them, <br/>
and He has **committed** to us the **word of reconciliation**.

---
## Outline for today
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
## Spinning-disk storage
<div class="imgbox"><div style="flex:3">
<ul>
<li> **Seek**: move head to **track**, <br/>
  wait for **sector** *(slow)*
<li> **Throughput**: read from <br/>
  consecutive sectors *(fast)*
<li> Lots of **small** *iops* (I/O ops/sec) are bad<ul>
  <li> So **buffer** and do I/O in larger *pages* at a time
  <li> Typical **page size** around *16KB*
</ul></ul>
</div><div>
![Hard disk, CHS](static/img/Fig-18-2.svg)
</div></div>

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
  + Also may be categorised by (Knuth) **order** = *2t*
+ All **leaves** are at same depth *h*
+ In terms of *t* and *h*, what is **min** num of keys stored?
  **Max**?
+ Variants: *B+*-tree: **payload** stored only in **leaves**
+ Variants: *B&lowast;*-tree: *2t-1* &lt; \`n\_k\` &lt; *3t-1*

---
## t=2 B-tree (2-3-4 tree)
+ In **B+**-tree, pointers to **data** go in leaf nodes

![2-3-4 B-tree](static/img/Fig-18-1.svg)

---
## Outline

---
## B-tree operations
+ **Search tree** interface: search, insert, delete
+ Assess not only **computational** complexity, but also
  + **Disk** accesses (read/write), in terms of *n* and *t*
+ Keep **root** in RAM (**write** to disk if modified)
  + Other nodes need to be **read** in from disk
+ Constraining **degree** (*t* .. *2t*) keeps tree **balanced**

![B-tree with t=1000](static/img/Fig-18-3.svg)

---
## B-tree search
```
def search( node, key ):

  # Linear search through keys
  for (i = 1; (i <= node.size) and (key > node.keys[i]); i++)

  if ((i <= node.size) and (key == node.keys[i])):
    return (node, i)        # found it!

  if (node.isLeaf()):
    return None             # key not in tree

  read( node.child[i] )     # load child from disk
  return search( node.child[i], key )       # recurse
```

+ **Tail** recursion can easily be changed to **loop**
+ **Compute** (worst-case): O( *th* ) = \`O(t log\_t n)\`
+ **Disk** accesses (worst-case): O(h) = \`O(log\_t n)\`

---
## B-tree insert
+ As in BST, **search** (down to leaf node)
+ As we go, **split** any full nodes (*2t-1* keys)
  to ensure space for insertion
  + Split: make **two** nodes with *t-1* keys each
  + Promote **median** key up a level
+ Once we reach **leaf** node, we have space to insert

![B-tree insert](static/img/Fig-18-5.svg)

---
## B-tree insert: example
<div class="imgbox"><div><ul>
<li> (a) **initial**: *t=3*
<li> (b) **non-full** leaf *ACDE*
<li> (c) full leaf *RSTUV*: **split**
<li> (d) **split** root *GMPTX*
<li> (e) **split** node *ABCDE*
</ul></div><div style="flex:4">
![B-tree insertion example](static/img/Fig-18-7.svg)
</div></div>

---
## Outline

---
## B-tree delete
+ **Descend** tree, ensuring each node has &ge; *t* keys
+ If key is **here**, and we're a **leaf**: just delete key
+ If key is **here**, and we're **not** a leaf:
  + If **left** child has &ge; *t* keys, replace key w/**predecessor**
  + If **right** child has &ge; *t* keys, replace key w/**successor**
  + Else, **merge** left+right children, and delete key
+ If key is **not** here (and we're not a leaf):
  + **Find** the child node the key ought to be in
  + If the **child** node has only *t-1* keys:
    + If either **sibling** has &ge; *t* keys, **steal** one
    + Else, **merge** with a sibling
  + **Iterate** into that child

---
## Delete: example
<div class="imgbox"><div><ul>
<li> (a) **initial**: *t=3*
<li> (b) **internal** node &ge; *t*, key in **leaf**
<li> (c) key in **internal**: use **predecessor**
<li> (d) key in **internal**: **merge** children
</ul></div><div style="flex:4">
![B-tree deletion, pt1](static/img/Fig-18-8-L.svg)
</div></div>

---
## Delete example, cont.
<div class="imgbox"><div><ul>
<li> (e) **internal** node *CL* too small,
  and **sibling** too small to steal from:
  **merge**
<li> (f) merge **pushes** *P* down from root
<li> (g) child *AB* too small: **steal** from *EJK*
<li> (d) key in **internal**: **merge** children
</ul></div><div style="flex:4">
![B-tree deletion, pt2](static/img/Fig-18-8-R.svg)
</div></div>

---
## B-tree summary
+ **Generalisation** of BST, but:
  + All **leaves** are at same **height** (*h* = *&Theta;(lg n)* )
  + **Degree** of each node is between *t* and *2t*
+ **Operations**:
  + **Create**: CPU *O(1)*, disk *O(1)*
  + **Search**/insert/delete: CPU *O(th)*, disk *O(h)*
+ When **modifying** tree, need to ensure **degree** of each node
  stays between *t* and *2t*
  + i.e., number of **keys** stored is between *t-1* and *2t-1*

---
## Using B-tree in filesystem
+ Filesystems store: **files**, **directories**, and **metadata**
  + e.g., name, owner, permissions, modification time)
+ File contents are stored in 1 or more **extents** on disk
  + i.e., Logical Block Addresses (LBA) interpretable by HDD
+ B-trees can be used for **lookup tables**:
  + **Inode** table: *metadata* for each object
    + Indexed by *inode*, unique to each object
  + **Directory** tables: list *files* in a directory
    + Map *filenames* (string) to *inodes*
  + **Extents** table: *LBAs* for each extent
    + Or the data itself, if small enough
  + **Journal**: transaction *log*
    + Preserve *integrity* in case a write fails

---
## Filesystems using B-trees
+ **NTFS** indexes (i.e., *inode* tables)
+ Mac **HFS** catalog records (i.e., *inode* tables) use **B+-trees**
+ Linux **ext3/4** directory indexes use **Htree** hash tables
  + **Hash** filenames for fast lookup
+ Linux **btrfs** ("B-tree filesystem") uses them everywhere:
  + **Directory** index (with hashed *filenames* )
  + **Extent** tree (payload is either *LBAs* or actual *data* )
  + **Log** tree (journal)
  + **Root** tree storing links to all the *other* trees!
  + and more...

---
## Outline

---
## Midterm review
+ Tue 25 Oct, **13:10-14:30** (80min)
  + **60 pts**, estimate 1min/pt
+ Open paper **book**, open paper **notes**
+ No **electronic** devices: computers, phones, etc.
  + Phone should be **off**/mute and in **pocket**/bag
+ Bring pen/pencil and **blank** paper to write on
+ TA will invigilate; he **cannot** answer questions on content
  + If you feel a question is **ambiguous**, write your interpretation
    on your exam sheet and answer accordingly
+ Mostly **hand-simulation**, but expect some **proofs** as well

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
