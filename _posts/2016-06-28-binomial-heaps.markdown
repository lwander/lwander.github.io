---
layout: post
title:  "Binomial Heaps"
date:   2016-06-28 12:18:01 -0400
categories: cs
---

Recently, I stumbled across the concept of _mergeable heaps_, which are simply
priority queues that provide a _merge_ operation. Alone, they aren't very
interesting; given a structure of elements with a total ordering
{% highlight sml %}structure elem : ORDERED{% endhighlight %}
and a type
{% highlight sml %}type heap{% endhighlight %}
they need to implement
{% highlight sml %}val new : heap
val insert : heap -> elem.t -> heap
val merge : heap -> heap -> heap
val findMin : heap -> elem.t
val deleteMin: heap -> heap
{% endhighlight %}

What makes them interesting is the combination of their uses (which we'll get
to in a later post), and the big-O time bounds that can be achieved for each
operation with a clever implementation.

A quick-and-dirty mergeable heap could be implemented as follows:
{% highlight sml %}datatype heap = Empty | Heap of elem.t * heap

(* O(n) time, where n is the size of the heap *)
fun insert Empty e = Heap (e, Empty)
fun insert (Heap (e', h)) e = 
    if Elem.lt e e' 
    then Heap (e, Heap (e', h))
    else Heap (e', insert e h)

(* O(min(m, n)) time, where n and m are the sizes of the 
 * input heaps *)
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
{% endhighlight %}

The time bounds can greatly improved with _binomeal heaps_, which are really
quite neat. A side by side comparison of the time bounds of each operation:

|             | List   | Binomeal Heap |
|-------------|-------:|--------------:|
| `insert`    | `O(n)` |        `O(1)` |
| `merge`     | `O(n)` |   `O(log(n))` |
| `findMin`   | `O(1)` |        `O(1)` |
| `deleteMin` | `O(1)` |   `O(log(n))` |

_implementation coming soon..._
