---
layout: default
---
# Learning from Book _Annotated STL Sources_ by **JJHou**
---

# Chapter 1: STL Introduction
- History of STL Evolution and Open Source Community
- A set of Compiler Macro Configuration for portability. 
- Techinque of **Generalization**, **Specilization** and **Refinement**
- Relationship between Components
  - Allocator
  - Container and Iterators
  - Algorithm
  - Functor
  - Adaptors

# Chapter 2: Allocator
## Construction and Destruction
- Global _construct()_ and _destroy()_ function for object construction and desctruction on the allocated memory
- Use refinement and specification technique to select proper construction implemenation based on whether having trivial destructor or not. 

## Two-level Memory Allocator
- Use _malloc()_ and _free()__ for large memory allocation
- Use pre-allocated **free lists** for small memory allocation 
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
  pop_heap_aux(first, last-1, last-1, e);


def pop_heap_aux(first, last, pos, e);
  *pos = *first;
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
* Efficient *push_front()*, *pop_front()*, *erase_after()* and *insert_after()* operations.
* Inefficient *push()*, *pop()* operations far from the head


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

## RB-Tree Powered Containers
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


# Chapter 6: Algorithms
## Numeric
- *accumulate()*
- *adjacent_difference()*
- *inner_product()*
- *partial_sum()*
- *power()*
```
def power(T x, uint n, MonoidOp op):
  if n == 0: return identitiy(op);
  while (n & 1) == 0:
    n >>= 1;
    x = op(x, x);

  T result = x;
  n >>= 1;
  while (n != 0):
    x = op(x, x);
    if (n & 1 == 1):
      result = op(result, x);
    n >>= 1;
  return result;
```

## Algobase
- *fill()*
- *fill_n()*
- *equal()*
- *iter_swap()*
- *max()*
- *min()*
- *mismatch()*
- *swap()*
- *compare()*
- *lexicographical_compare()*

## Copy
- Any Input Iterator (Generalization)
  - Refinement for forward iterator
    - Sequetial copy by advancing source and destination iterators
  - Refinement for random access iterator
    - Sequential copy from i = 0 to number of elements   
  - T\* (partial specification)
  - const T\* (partial specification)
    - has trivial operator=
      - use *memmove()*
    - No trivial opererator=
      - Treat as random access iterator
- const char\* (specification)
  - use *memmove()*
- const wchar_t\* (specification)
  - use *memmove()*
- *copy_backward()* (Similarly handled as above)

## Set
- *set_union()*
- *set_intersection()*
- *set_difference()*
- *set_symmetric_difference()*

## Others
- *adjacent_find()*
  - Return the iterator that points to the first element of the first adjacent equal pair
- *count()*
- *count()_if()*
- *find()*
- *find_if()*
- *search_n()*
  - Find the first continuous occurrence of an element inside another interval
- *search()*
  - Find the first occurrence of an subinterval inside another interval
- *find_end()*
  - Find the last occurrence of an subinterval inside another interval
  - Refinement on Forward Iterator
    - Use *search()* to continuously search for the interval until the end
  - Refinement on Bidirectional Iterator
    - Search backward for the occurence of subinterval
- *find_first_of()*
  - Find the first occurrence of any element in subinterval inside another interval
- *generate()*
- *generate_n()*
- *includes()*
  - Return whether the parent interval contains all the elements in a subinterval while the elements in subinterval is preserved. 
- *max_element()*
- *min_element()*
- *merge()*
- *partition()*
  - Partition the interval based on a predicate
  - Return the iterator that points to the first element non-qualified element
  - Original order is not preserved (Non-stable)
- *remove()*
- *remove_copy()*
- *remove_copy_if()*
- *replace()*
- *replace_copy()*
- *replace_if()*
- *replace_copy_if()*
- *reverse()* (Bidirectional Iterator Only)
- *reverse_copy()*
- *rotate()*

```
// Refinement for Forward Iterator
def rotate(ForwardIterator f, m, l):
  Iterator i = m;
  while(1):
    iter_swap(f, i);
    ++f;
    ++i;
    if f == m:
      if i == l return;
      middle = i;
    else if i == l:
      i = middle;

// Refinement for Bidirectional Iterator
def rotate(BidirectionalIterator f, m, l):
  reverse(f, m);
  reverse(m, l);
  reverse(f, l);

// Refine for Random Access Iterator
def rotate(RandomAccessIterator f, m, l):
  g = gcd(m - f, l - m);
  for i from 0 to g-1:
    o = i;
    // cyclic rotate
    do:
      n = (o - (m - f)) mod (l - f);
      move element from o to n;
      o = (o + (m - f)) mod (l - f);
    while (i != o)
```

- *rotate_copy()*
- *swap_ranges()*
- *transform()*
- *unique()*
- *unique_copy()*

## Sorted Interval Only 
- *lower_bound()*
  - Return the iterator that points to the first element no smaller than k
- *upper_bound()*
  - Return the iterator that points to the first element bigger than k
- *equal_range()*
  - Return an pair of *lower_bound()* and *upper_bound()*
- *binary_search()*

```
def lower_bound(Iterator f/, e, k):
  len = e - f;
  while len > 0:
    m = f + len >> 1;
    if *middle < k:
      f = m + 1;
      len -= half + 1;
    else:
      len = half;
  return f;

def upper_bound(Iterator f, e, k):
  len = e - f;
  while len > 0:
    m = f + len >> 1;
    if *middle <= k:
      f = m + 1;
      len -= half + 1;
    else:
      len = half;
  return f;

def equal_range(Iterator f, e, k):
  len = e - f;
  while len > 0:
    m = f + len >> 1;
    if *middle < k:
      f = m + 1;
      len -= half + 1;
    else if k < *middle:
      len = half;
    else 
      lower = lower_bound(f, m);
      upper = upper_bound(m + 1, e)
      return (lower, upper);
  return (first, first);
```

## Permutation and Shuffle
### Permutation Order (From sorted to reverse sorted)
1. (1, 2, 3)
1. (1, 3 ,2)
1. (2, 1, 3)
1. (2, 3, 1)
1. (3, 1 ,2)
1. (3, 2, 1)

- *next_permulation()*/*prev_permulation()*
  - Bidirectial Iterator
  - Given a permulation, return the next/previous one in the above order
  - Return false if it is cyclic to start/end. True, otherwise
- *random_shuffle()*
  - Random Access Iterator

```
def next_permutation(BidirectionalIterator first, last):
  Return false if 0 or 1 element. 
  i = last;
  while 1:
    ii = i;
    i = ii - 1;
    if *i < *ii:
      // Find from the end the first adjacent pair of the first element smaller than the second element 
      j = last;
      while !(*i < *--j);
      // Until find j from the end such that *i >= *j
      iter_swap(i, j);
      reverse(ii, last);
      return true;

    if first == last:
      reverse(first, last);
      return false;


def prev_permutation(BidirectionalIterator first, last):
  Return false if 0 or 1 element. 
  i = last;
  while 1:
    ii = i;
    i = ii - 1;
    if *ii < *i:
      j = last;
      while(!(*--j < *i));
      // Until find j from the end such that *i >= *j
      iter_swap(i, j);
      reverse(ii, last);
      return true;

    if first == last:
      reverse(first, last);
      return false;
```

## Sort
### Partial Sort
```
// Sort the first m - f elements between [f, m)
def partial_sort(RandomAccessIterator f, m, l):
  make_heap(f, m);
  for i = from m to l - 1:
    if (*i < *f)
      pop_heap_aux(f, m, i, *i);
  sort_heap(first, middle);
```

### Insertion Sort
Good Performance for nearly-sorted interval or small interval
```
def insertion_sort(RandomAccessIterator f, l):
  if f == l return
  for i from f+1 to l-1:
    linear_insert(l, i, *i);

def linear_insert(RandonAccessIterator f, l, Element e):
  if e < *f
    copy_backward(f, l, f + 1);
    *first = e
  unguarded_linear_insert(f, l, e);

// unguarded means eventually, there must exist an element smaller than e.
//   So no need to worry out of bound. 
def unguarded_linear_insert(RandonAccessIterator f, l, Element e):
  --l;
  while e < *l:
    *(l+1) = *l;
    --l;
  *(++l) = e;

```

### Merge Sort
```
def merge_sort(BidirectionalIterator f, l):
  len = f - l;
  m = f + len / 2;
  merge_sort(f, m);
  merge_sort(m+1, l);
  merge_adaptive(f, m, l);

// Sorted subinterval [f, m) and [m+1, l) to be merged on [f, l)
//   e.g, [1, 3, 5] and [2, 4]
def merge_adaptive(BidirectionalIterator f, m, l):
  len = l - f;
  l1 = f - m
  l2 = l - m;
  buffer = request a buffer of len
  if (l1 < l2 && l1 < buf_size):
    copy [f, m) to buf
    merge buf and [l, m-1) to f
  else if (l2 < buf_size):
    copy [m+1, l) to buf
    merge backward buf and [f, m-1) to f
  else:
    // buf_size < l1 , l2
    // assume l1 < l2
    Partition [m+1, l) to equally two lists l21, l22
    Partition [l, m) to l11, l12 such that all l11 elements are smaller or equal to all elements of l22
    [l11, l12, l22, l21]
    Rotate [l12, l22] to give [l11, l22, l12, l21] with buffer
    merge_adaptive(l11, l22) with buffer;
    merge_adaptive(l12, l21) with buffer;
```

### Intro Sort
- Implementation of std::sort
  - For small or near-sorted interval, switch to insertion_sort
  - Recursively partition the interval as in quick sort until the interval length is smaller than threshold. 
  - For too deep recursion level, use heap sort

```
def sort(RandomAccessIterator f, l):
  intro_sort(f, l, MAX_LEVEL);
  final_insertion_sort(f, l);

def intro_sort(f, l, level):
  len = l - f;
  while l - f > THRESHOLD:
    if level == 0: 
      partial_sort(f, l, l);  // heap sort actually
    m = f + len / 2;
    med = median(*f, *m , *l);
    cut = partition(f, l, med);
    intro_sort(m+1, l, level-1);
    l = cut;

def final_insertion_sort(f, l):
  len = l - f;
  if len > THRESHOLD:
    insertion_sort(f, f + THRESHOLD);
    for i from f + TREHSHOLD to l - 1:
    // Unguarded because there exsists an element in the first THRESHOLD elements smaller than *i due to the previous partitioning
      unguarded_linear_insert(f, i, *i);
  else:
    insertion_sort(f, l);
```

### nth_element
- Function
  - nth iterator will contain the element which shall reside in nth if sorted
  - All elements before nth is smaller or equal to \*nth
  - All elements after nth is greater or equal to \*nth
- Algorithm
  - Recursive partition the interval which contains nth
  - Insertion sort the interval when the interval length is smaller than a threshold
```
def nth_element(RandomAccessIterator f, l, nth):
  while l - f > 3:
    m = f + (m - f) / 2;
    med = median(*f, *m, *l);
    cut = partition(f, l, med);
    if cut <= nth:
      f = cut;
    else:
      l = cut;
  insertion_sort(f, l);
```

# Chapter 7: Functor
## Inherit the following template to make self-defined functor adaptable
- ```unary_function<typename Arg, typename Result>```
- ```binary_function<typename Arg1, typename Arg2, typename Result>```

## Functor Classification
- Arithmetic
  - *plus\<T\>*
  - *minus\<T\>*
  - *multiplies\<T\>*
  - *divides\<T\>*
  - *modulus\<T\>*
  - *negate\<T\>*
  - *identity_element\<T\>*
- Relational
  - *equal_to\<T\>*
  - *not_equal_to\<T\>*
  - *greater\<T\>*
  - *greater_equal\<T\>*
  - *less\<T\>*
  - *less_equal\<T\>*
- Logical
  - *logical_and\<T\>*
  - *logical_or\<T\>*
  - *logical_not\<T\>*
- Others
  - *identity\<T\>*
  - *select1st\<Pair\>*
  - *select2st\<Pair\>*
  - *project1st\<Arg1, Arg2\>*
  - *project2nd\<Arg1, Arg2\>*

## Chapter 8: Adaptor
### Container Adaptor
* stack
* queue

### Iterator Adaptor
#### Inserter Iterator
```
class BackInserter<Container> {
  typedef output_iterator_tag iterator_category;
  Container* c;
  BackInserter<Container>& operator=(Element e) {
    c->push_back(e);
    return *this;
  }

  BackInserter<Container>& operator*() {
    // Unlike normal iterator that returns the reference to the pointed element,
    // it returns itself the reference. 
    return *this; 
  }

  BackInserter<Container>& operator++() {
    // This is to be compatible for incremental assignment operation, such as
    //   *(++back_inserter) = *(++input_iterator);
    return *this;
  }

  BackInserter<Container>& operator++(int) {
    // Same rationale as above
    return *this;
  }
};

class FrontInserter<Container> {
  // Similarly defined as above
  // Use push_front during assignment operation
}

class Inserter<Container, Iterator> {
  // Similarly defined as above
  // Use insert during assignment operation
}
```
- Back Inserter

#### Reverse Iterator
```
class ReverseIterator<Iterator> {
  typedef Iterator::iterator_category iterator_category;
  Iterator base;
  Iterator::reference operator*() {
    // There exists one offset between normal iterator. 
    return *(base-1);
  }
}
```
#### Stream Iterator
- istream_iterator<T>
- ostream_iterator<T>

```
class istream_iterator<T> {
  typedef input_iterator_tag iterator_category;
  T value
  istream_iterator<T>(istream s) {
    // Read in the first value during construction
    s >> value;
  }

  T& operator*() {
    return value;
  }

  istream_iterator<T>& operator++() {
    s >> value;
    return this;
  }
  istream_iterator<T>& operator++(int) {
    tmp = this;
    // Read in the next value
    s >> value;
    return tmp;
  }

}

class ostream_iterator<T> {
  typedef output_iterator_tag iterator_category;
  // Similarly handled with all output iterators
  ostream_iterator<T>& operator*() {
    return *this;
  }

  ostream_iterator<T>& operator++() {
    return *this;
  }

  ostream_iterator<T>& operator++(int) {
    return *this;
  }
}
```

### Functor Adaptor
- *not1()*
  - ```unary_negate<class Predicate>```
- *not2()*
  - ```binary_negate<class Predicate>```
- *bind1st()*
  - ```binder1st<class Operation>```
- *bind2nd()*
  - ```binder2nd<class Operation>```
- *compose1()*
  - ```unary_compose<class Op1, class Op2>```
- *compose2()*
  - ```binary_compose<class Op1, class Op2, class Op3)```
- *ptr_fun()*
  - ```pointer_to_unary_function<Arg, Result>```
  - ```pointer_to_unary_function<Arg1, Arg2, Result>```
- *mem_fun()*
  - ```mem_fun_t<T*, Result>```
  - ```const_mem_fun_t<T*, Result>```
  - ```mem_fun1_t<T*, Arg, Result>```
  - ```const_mem_fun1_t<T*, Arg, Result>```
- *mem_fun_ref()*
  - ```mem_fun_ref_t<T*, Result>```
  - ```const_mem_fun_ref_f<T*, Result>```
  - ```mem_fun1_ref_t<T*, Arg, Result>```
  - ```const_mem_fun1_ref_t<T*, Arg, Result>```






[back](./)
