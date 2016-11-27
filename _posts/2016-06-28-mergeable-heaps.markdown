---
layout: post
title: "Mergeable Heaps"
date: 2016-06-28 12:18:01 -0400
categories: cs
---

Recently, reading [Purely Functional Data
Structures](https://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504)
I stumbled across the concept of _mergeable heaps_, which are simply
priority queues that provide a `merge` operation. Alone, they aren't very
interesting; given a structure of elements with a total ordering

```
structure elem : ORDERED
```

and a type

```
type heap
```

they need to implement

```
val new : heap
val insert : heap -> elem.t -> heap
val merge : heap -> heap -> heap
val findMin : heap -> elem.t
val deleteMin: heap -> heap
```

What makes them interesting is the combination of their uses (which we'll get
to in a later post), and the big-O time bounds that can be achieved for each
operation with a clever implementation.

## List Heaps

A quick-and-dirty mergeable heap, called a _list heap_, could be implemented
as follows:

```
datatype heap = Empty | Heap of elem.t * heap

(* O(n) time, where n is the size of the heap *)
fun insert Empty e = Heap (e, Empty)
fun insert (Heap (e', h)) e =
    if Elem.lt e e'
    then Heap (e, Heap (e', h))
    else Heap (e', insert e h)

(* O(min(m, n)) time, where n and m are the sizes of the input heaps *)
fun merge Empty h = h
fun merge h Empty = h
fun merge (Heap (e , h)) (Heap (e', h')) =
    if Elem.lt e e'
    then Heap (e, merge h (Heap (e', h')))
    else Heap (e', merge (Heap (e, h)) h')

(* O(1) time *)
fun findMin Empty = raise EMPTY
fun findMin Heap (e, _) = e

(* O(1) time *)
fun deleteMin Empty = raise EMPTY
fun deleteMin Heap (_, h) = h
```

As you can see, these are unimaginative and slow. If we start to
think about how we can exploit the fact that we are maintaining the order of
the elements in our list, we could improve the performance of `insert` to
worst-case constant time using Dietz & Sleator's [order
maintenance](http://www.cs.cmu.edu/~sleator/papers/maintaining-order.pdf) data
structure. `merge` will naively still take \\(O(n)\\) time since we can rebuild
both lists with a series of \\(O(n)\\) `insert`s. However, order maintenance
data structure is a bit heavy-handed in this case, since it permits arbitrary
`delete` in constant time, when all we are interested in is `deleteMin`. So
let's take a look at how else can start to improve these time bounds.

## Binomeal Heaps

The time bounds can improved with _binomeal heaps_ (and later
_fibonacci heaps_), which are really neat. A side by side comparison of
the amortized time bounds of each operation:

|             | List/OM Heap  | Binomeal Heap     | Fibonacci Heap     |
|-------------|--------------:|------------------:|-------------------:|
| `insert`    | \\(O(1)\\)    |        \\(O(1)\\) |          \\(O(1)\\)|
| `merge`     | \\(O(n)\\)    |   \\(O(log(n))\\) |          \\(O(1)\\)|
| `findMin`   | \\(O(1)\\)    |        \\(O(1)\\) |          \\(O(1)\\)|
| `deleteMin` | \\(O(1)\\)    |   \\(O(log(n))\\) |      \\(O(log(n))\\)|

To understand how binomeal heaps work, we need to first examine _binomial
trees_, which we will find are useful in this post and in a few to come.
To begin, binomial trees are categorized by their rank, a non-negative
integer \\(r\\). A binomial tree of rank 0 is simpy a single node. A binomial tree of
rank \\(r > 0\\) has \\(r\\) children, \\(t_1, t_2, ..., t_r\\), such that each
tree \\(t_i\\) has a rank of \\(r - i\\). (The order matters here, as I will
be referring to the subtrees as right-most and left-most). A picture is very
helpful here:

```
Rank 0    Rank 1    Rank 2    Rank 3

  .         .         .       --.
            |        /|      / /|
            .       . .     . . .
                    |      /| |
                    .     . . .
                          |
                          .

```

Now it should be a little easier to see the pattern described above. Now notice
how for a binomial tree of rank \\(r\\), it's largest subtree is of rank
\\(r - 1\\). This is valuable, because if we are given two binomal trees of
rank \\(r\\), simply making the left-most child of one tree the other tree, we
have a binomial tree of rank \\(r + 1\\). Furthermore, notice how this implies
that the number of nodes in a binomial tree of rank \\(r\\) is directly
\\(2^r\\).

With these two pieces of information, and a little creativity, we can encode
bitstrings as lists of binomial trees, such that for a bitstring \\(b\\), we can say
that if the \\(i^{th}\\) bit \\(b_i\\) is set, our list of binomial trees contains
a tree with rank \\(i\\). It's easy to see that the total number of nodes will
equal the number the bitstring encodes. Furthermore, we can define addition
on these lists of binomial trees in an efficient matter if we keep the lists
sorted by increasing rank. We simply merge the two lists, and whenever we
encounter two trees of equal rank, we `link` them, and carry the increased rank
tree forward. Let's take a look at how this is implemented:

```
(* The int is the rank, unit is a placeholder for later, and the list is the
 * ordered set of children the tree owns *)
datatype Tree = Node of int * unit * Tree list

fun rank (Node (r, _, _)) = r

fun link (t1 as Node (r1, _, l1)) (t2 as Node(r2, _, l2)) =
    if r1 <> r2 then raise InvalidArgument
    else Node (r1 + 1, (), t2::l1)

fun carry t [] = [t]
fun carry t (t'::tl') =
    if rank t < rank t' then t::t'::tl'
    else carry (link t t') tl

fun add tl [] = tl
fun add [] tl = tl
fun add (t1::tl1) (t2::tl2) =
    if rank t1 < rank t2 then t1::(merge tl1 (t2::tl2))
    else if rank t2 < rank t1 then t2::(merge (t1::tl1) tl2)
    else merge (carry t2 (t1::tl1)) tl2
```

So this is clearly all very neat, but as-is, useless. We just defined
some of the operations needed to add non-negative integers, where each
integer n takes \\(O(n)\\) space. But, this becomes interesting once we redefine
our `Tree` datatype and `link` function to be

```
datatype Tree = Node of int * Elem.t * Tree list

fun value (Node (_, x, _)) = x

fun link (t1 as Node (r1, x1, l1)) (t2 as Node(r2, x2, l2)) =
    if r1 <> r2 then raise InvalidArgument
    else if Elem.lt x1 x2 then Node (r + 1, x1, t2::l1)
    else Node(r + 1, x2, t1::l2)

fun merge = add

fun insert t x = merge [(Node (0, x, []))] t
```

What happens now is that during `add` operations, we always preserve
the smallest element at the root of each binomial tree in the list with
constant time additional work. Also, we got `merge` for free, since it's
merging two binomeal heaps is effectively adding their binary representations,
and inserting a new element simply requires adding a new singleton tree into
our existing heap. We can argue that the amortized insertion time is constant,
by the classic amortized binary counter argument.

What's left is `findMin` and `deleteMin`. Keeping in mind that our binomial
heap's list of trees are all rooted by their smallest element, we do a simple
linear search:

```
fun findMin [] = raise InvalidArgument
fun findMin (Node (_, x, _))::[] = x
fun findMin (Node (_, x, _))::ts =
    let
        val x' = findMin ts
    in
        if Elem.lt x x' then x else x'
    end
```

`deleteMin` is a bit trickier, but really neat:

```
fun deleteMinTree [] = raise InvalidArgument
fun deleteMinTree t::[] = (t, [])
fun deleteMinTree (t::ts) =
    let
        val (t', ts') = deleteMinTree ts
    in
        if Elem.lt (value t) (value t') then (t, ts) else (t', t::ts')
    end

fun deleteMin [] = raise InvalidArgument
fun deleteMin t =
    let
        val (Node (_, _, ts'), ts) = deleteMinTree t
    in
        merge (rev ts') ts
    end
```

Keep in mind that the branches of a binomial heap are a rank-reverse binomial
heap, so that `merge` produces a valid binomial heap with a single element
missing.

## Fibonacci Heaps

You'll notice that the biggest cost in the _binomial heap_ implementation was
the fact that for any merge or insertion we might have to do \\(O(log(n))\\) 
comparisons (along the \\(O(log(n))\\) binomial tress in the list). 

_More coming soon..._
