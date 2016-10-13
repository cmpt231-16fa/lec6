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
## Outline

---
<!-- .slide: class="empty" -->
