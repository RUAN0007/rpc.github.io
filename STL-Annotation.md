---
layout: default
---
# Learning from Book _Annotated STL Sources_ by **JJHou**
---

# Chapter 1: STL Introduction
## History of STL Evolution and Open Source Community
## A set of Compiler Macro Configuration for portability. 
## Techinque of **Generalization**, **Specilization** and **Refinement**
## Relationship between Components
* Allocator
* Container and Iterators
* Algorithm
* Functor
* Adaptors

# Chapter 2: Allocator
## Global _construct()_ and _destroy()_ function for object construction and desctruction on the allocated memory
### Use refinement and specification technique to select proper construction implemenation based on whether having trivial destructor or not. 

## Two-level Memory Allocator
### Use _malloc()_ and _free()__ for large memory allocation
### Use pre-allocated **free lists** for small memory allocation 
If the free list of desirable size is exhausted, refill it using larger memory chunk or request using _malloc()_. 

## Global functions to copy or fill objects on allocated but uninitiatized memory
* *uninitialized_copy()*
* *uninitialized_fill()*
* *uninitialized_fill_n()*

# Chapter 3: Iterators and Traits
## Five corresponding types for an iterator.
* iterator_category
* value_type
* difference_type
* pointer
* reference

 **NOTE**: pointers or const pointers are also iterators. Partial specification for these pointers on iterator traits defines their five corresponding types. Any self-defined iterators must typedef the above types for compatible use with STL other components.


## Iterator herarchical catorgories and tags:
* Output Iterator
* Input Iterator
* Forward Iterator: Only allow for forward movement
* Bidirectional Iterator: Allow for both forward and backward movement
* RandomAccess Iterator: Allow for random access

# Chapter 4: Sequence Containers
## Vector
* A single chunk of continuous memory
* Random access iterators (a pointer)
* Constant time for back modification (insertion, removal) but linear time for front and middle modification
* If the new element number exceeds the reserved size, a larger memory is requested and the entire vector will be copied there.

## List
* A double-linked node list with an empty node at the end
* Bidirectional iterator
* Const time for front, middle and back modification
* The list contains the last empty node in the node list
* Internal algorithm for sorting (A modified version of merge sort that support up to 2^64 elements)
```
list sort(List l):
  List r; // The final sorted list
  List tmp;
  List ls[64];  // Initialize 64 empty lists
  int maxIdx = 0; // The largest index for non-empty lists

  for each element e in l:
    i = 0;
    l = [e];
    while (ls[i] is not empty)
      l = merge(l, ls[i]);
      ls[i] = [];
      ++i;
    ls[i] = l; l = []; // or swap(ls[i], l);
    if i == maxIdx  ++maxIdx;

  for i from 0 to maxIdx;
    r = merge(r, ls[i]);
  return r;
```

## Deque
- A list of chunks of continuous memory with the same size
  - The deque internally maintains an array of chunk memory pointers, referred as central controllers. 
- Random access iterator consisting of the four pointers
  - A pointer to the chunk start
  - A pointer to the chunk end
  - A pointer to the pointed element
  - A pointer to an array of memory pointers
- Constant time for front and back modification and linear time for middle modification
  - If front/end modification exceeds the first/last chunk, a new chunk will be allocated or the existing chunk will be deallocated. 
  - The controller will be updated accordingly. Controller elements will be placed in the middle of the controller array. 
  - If the new controller element is reaching the current array head or tail, elements will be readjusted to middle. Possibly, a larger array is requested and copied to. 

## Stack/Queue
- Limited access on a sequence of elements
  - NO iterator defined. 
- Adaptors on a generic sequence, default to be _deque_. 

## Heap
- A complete binary tree with _parent node > each of child node_ (Max Heap)
- No iterators
- Define a set of global functions to work on an interval of two random access iterators.
  - *push_heap()*
  - *pop_heap()* 
  - *sort_heap()*
  - *make_heap()*

```
/*  Function: Percolate up the element to adjust a heap
    Precondition: 
      1. The element to be pushed is at last - 1. 
      2. Interval [first, last-1) forms a valid heap. */

def push_heap(first, last):
  push_heap_aux(first, last - first - 1, 0, *(last-1));

def push_heap_aux(Iterator first, Distance holeIdx, 
                  Distance topIdx, Element value):
  Distance parent = (holeIdx - 1) / 2;
  while (holeIdx > topIdx AND *(first + parent) < value):
    Iterator hole = first + holeIdx;
    swap(first + parent, first + holeIdx);
    holeIdx = parent;
    parent = (holeIdx - 1) / 2;
  *(first + holeIdx) = value;
```

``` 
/*  Function: Pop the max element to (last - 1) and maintain [first, last - 1) as a heap
    Precondition:
      1. Interval [first, last) form a heap. */

def pop_heap(first, last):
  Element e = *(last - 1);
  *(last - 1) = *first;
  adjust_heap(first, 0, (last - first - 1), e);
```

```
/*  Function: Percolate down for the new element for the hole 
    Precondition:
      1. Each subtree of the hole form a valid heap. */

def adjust_heap(Iterator first, Distance holeIdx, 
                Distance len, Element e):
  Distance topIdx = holeIdx;
  Distance second_child = 2 * holeIdx + 2;
  while (second_child < len):
    if (*(first + second_child - 1) < *(first + second_child)):
      --second_child;
    *(first+holeIdx) = *(first + second_child);
    holeIdx = second_child;
    second_child = 2 * holeIdx + 2;
  if (second_child == len):
    *(first + holeIdx) = *(first + second_child - 1);
    holeIdx = second_child - 1;
  push_heap_aux(first, holeIdx, topIdx, value);
```

```  
/*  Function: Sort an existing heap
    Precondition: 
      1. Interval [first, last) form a heap. */

def sort_heap(first, last):
  while(last - first > 1):
    pop_heap(first, last);
    --last;
```

```
/* Function: Adjust an interval to a valid heap
def make_heap(first, last):
  len = last - first;
  // The last element with a child.
  parent = first + len / 2 - 1; 
  while (parent != first):
    adjust_heap(first, parent, len, *(first + parent));
    --parent;

```

## Priority Queue
* Adaptors on Heap
* No iterator

## SList
* A singly-linked list
* Forward Iterator
* Efficient _push_front()_, _pop_front()_, _erase_after() and _insert_after_() operations.
* Inefficient _push()_, _pop()_ operations far from the head


# Chapter 5: Associative Containers
## Definition of Binary Tree
-    left child key <= parent key < right child key
- AVL Tree
  - The height difference between left and right subtree is at most 1.
  - Frequent Rotation to balance the tree
- Red Black Tree
  - Each node is either Red or Black. 
  - Each path from from root to leaf shares the same number of black nodes. 
  - Black root node
  - No adjacent red nodes. 
  - Less frequent rotation to balance the tree

## RB Tree
### Tree Node Fields and Functions
* Value field
* Pointer to parent node
* Pointer to left node
* Pointer to right node
* Functions to retrieve leftmost child node
* Functions to retrieve rightmost child node

### Tree Iterator
* Bidirectional Iterator

### Tree Structure
A RB-Tree contains a fake header node, which
* points to root node as parent,
* points to leftmost node as left child,
* points to rightmost node as right child,
* is pointed by root node as parent.

- Modification
  - *insert_unique()*
    - return a pair of iterator and bool
      - The iterator points to inserted or existed element
      - bool for true for successful insertion. False, otherwise. 
  - *insert_equal()*
    - return a single iterator
- Query
```
iterator search(Key k, Node header):
  y = header;  // the smallest node which is no smaller than k. 
  x = header->parent(); // root
  while (x ! = 0):
    if (k <= x):
      y = x;
      x = x.left();
    else:
      x = x.right();
  iterator j(y);
  if j == end() || x < y:
    return end(); // Not found
  else 
    return j;
```

## RB-Tree Powered Contaioners
- _map_/_set_
  - _insertion()_
    - Use RB Tree *insert_unique()* for insertion
  - _operator[](const K& k)_
    - Insert a default fake value and then return the iterator reference 
    - ```return (*((insert(value_type(k, T()))).first).second'; ```
- *multi_map*/*multi_set*
  - _insertion()_
    - Use RB Tree *insert_equal()* for insertion

## Introduction to Hash Table
- Loading Factor (LF)
- Linear Probing (LF < 1)
- Quadratic Probing (LF < 1)
- Seperate Chaining (LF > 1)

## Hash Table
### Hash Table Node
- Value field
- Pointer to the next node

### Iterator
- Forward Iterator

### Hashtable Structure
Maintain a vector of linked list of nodes
- Each element is chained to the list, which is determined by the hash of the key of the element. 
- The size of the vector is equal to the smallest prime number no smaller than the element number of hash table. 
- If the number of elements exceeds the vector size, another larger prime-number-element vector shall be requested and replaced the original one. 

- Modification (Similar to RB Tree)
  - *insert_unique()*
  - *insert_equal()*

### Hashtable-powered Data Structure
- *hash_map*/*hash_set*
  - use *insert_unique()*
- *hash_multimap*/*hash_multiset*
  - use *insert_equal()*

[back](./)
