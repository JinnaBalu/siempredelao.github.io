---
layout: post
title: "About Java Collections framework interfaces..."
author: David Aguiar
date: 2016-09-29 12:00:00
categories: Java
highlight: true
comments: true
image: /img/welcome_card.jpg
---
> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually* written in Markdown! To get a feel for Markdown's syntax, type some text into the left window and watch the results in the right.


```sh
127.0.0.1:8000
```


The Collections framework in Java is ... and something that all Java developers should master.
During last days, I have been reading about collections in Java. Here I show you some basic notes I wrote down which I consider the most important.


The Java Collections Framework hierarchy consists of two different interface roots:
- Collection interface
- Map interface

(show image with the core collection interfaces)

### Collection

The Collection interface represents a group of elements of a specific type. It is the least common denominator for all Collection implementations and defines a group of methods that all collections should implement (although if a method is not needed, it can throw `UnsupportedOperationException`). The Collection interface is generic (Collection<E>), so we are forced to specify its type of object in any implementation. That way, types are checked at compile-time, showing errors if types do not match.

#### Set

A `Set` is a Collection implementation which contains unique elements.

##### SortedSet

A Set sorted by ascending order or elements' natural ordering or any specific one using a Comparator. It offers range-view, endpoint access and comparator access. There is no easy way to go into the interior of the SortedSet and iterate backward. Unlike the List interface, a range-view of a sorted set is really just a window onto whatever portion of the set lies in the designated part of the element space. That is, in List we get a sublist by index, so if we change the main list, indexes will change.

##### General purpose implementations

There are 3 general-purpose implementations: 
 - HashSet: stores Set elements in a hash table, not guaranteeing the order. Best-performing.
 - TreeSet: stores Set elements in a red-black tree, ordered by value. Least-performing.
 - LinkedHashSet: stores Set elements in a hash table with a linked list running through it, ordered by insertion-order. Slightly less performing than HashSet.

#### List

A `List` is a Collection implementation with ordered (by index) elements (duplicates or not). It offers positional access, search, iteration and range-view.

##### General purpose implementations

There are 2 general-purpose implementations: 
- ArrayList: a List implementation with a dynamically resizable array. It usually is the best-performing. The performance of removing elements from the end of the list is substantially better than that of removing elements from the beginning.
- LinkedList: doubly-linked list, implementing List and Deque interfaces. Better performance under certain circumstances.

#### Queue

Every Queue implementation must specify its order (could be FIFO, by priority,...) and could be limited in size. This interface does not provide blocking methods, see BlockingQueue instead.

##### General purpose implementations

There is only one general-purpose implementation: 
- PriorityQueue: a Queue implemented as a priority heap. Elements are ordered by natural ordering or any specific order using a Comparator.

#### Deque

A `Deque` is a linear collection of elements that supports the insertion and removal of elements at both end points, so it behaves as stack and queue. In other words, it represents a double-ended queue.

##### General purpose implementations

There is only one general-purpose implementation:
- ArrayDeque:

### Map

A `Map` is an object which maps unique keys to at most one value. If you need to map a key with more than one value, please use a Multimap. Two Map instances are equal if they represent the same key-value mappings. Although it is not a Collection per se, it contains some methods to be seen as a Collection (keySet, entrySet, values), where we can iterate to remove but not to add new elements.

##### General purpose implementations

There are 3 general-purpose implementations:
 - HashMap
 - TreeMap
 - LinkedHashMap
, with behaviour and performance analogous to HashSet, TreeSet and LinkedHashSet, although a LinkedHashSet can also be ordered by access-order (that is, from least-recently accessed to most-recently).
- SortedMap: a Map sorted by ascending order or keys' natural ordering or any specific one using a Comparator. It offers range-view, endpoint access and comparator access.



Add "Interesting Facts" on each one?
Add some algorithms?
Add a use case for each one?

So that's enough for today! In this introductory post, I have covered the most used Java Collection interfaces and general purpose implementations definitions. Apart of these ones, there are more implementations but I decided to not write about them because it would be the never ending story (and it is already perfectly explained in the Java documentation).


### Want to know more?

- [Java tutorial] [tutorial]

**PS: don't repeat yourself (DRY)! There are many polymorphic algorithms already available in the _Collections_ class, so check first if the functionality you need is already there! ;)**



[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job)

   [tutorial]: <https://docs.oracle.com/javase/tutorial/collections/interfaces/index.html>
