---
layout: post
title: "About Java Collections framework interfaces..."
author: David Aguiar
date: 2016-10-24 08:00:00
categories: Java
highlight: true
comments: true
image: /img/collections_image.png
---

The Collections framework in Java is an esential part of the Java platform used to manage groups of elements of the same type and it is also something that all Java developers should master.
During last days, I have been reading about this framework and taking some important basic notes I want to present you in this post. Let's go!

The Java Collections Framework hierarchy consists of two different interface roots:
- Collection interface
- Map interface

![diagram]

### Collection interface

The Collection interface represents a group of elements of a specific type. It is the least common denominator for all Collection implementations and defines a group of methods that all collections should implement (although if a method is not needed, it can throw `UnsupportedOperationException`). The Collection interface is generic (Collection\<E\>), so we are forced to specify its type of object in any implementation. That way, types are checked at compile-time, showing errors if types do not match.

#### Set

A `Set` is a Collection implementation which contains unique elements.

##### SortedSet

A Set sorted by ascending order or elements' natural ordering or any specific one using a Comparator. It offers range-view, endpoint access and comparator access. There is no easy way to go into the interior of the SortedSet and iterate backward. Unlike the List interface, a range-view of a sorted set is really just a window onto whatever portion of the set lies in the designated part of the element space. That is, in List we get a sublist by index, so if we change the main list, indexes will change.

##### General purpose implementations

There are three general-purpose implementations: 
 - HashSet: stores Set elements in a hash table, not guaranteeing the order. Best-performing.
 - TreeSet: stores Set elements in a red-black tree, ordered by value. Least-performing.
 - LinkedHashSet: stores Set elements in a hash table with a linked list running through it, ordered by insertion-order. Slightly less performing than HashSet.

#### List

A `List` is a Collection implementation with ordered (by index) elements (duplicates or not). It offers positional access, search, iteration and range-view.

##### General purpose implementations

There are two general-purpose implementations: 
- ArrayList: a List implementation with a dynamically resizable array. It usually is the best-performing. The performance of removing elements from the end of the list is substantially better than that of removing elements from the beginning.
- LinkedList: doubly-linked list, implementing List and Deque interfaces. Better performance under certain circumstances.

#### Queue

A `Queue` is a Collection implementation which hold elements prior to processing. Every Queue implementation must specify its order (could be FIFO, by priority,...) and could be limited in size. This interface does not provide blocking methods, see BlockingQueue instead.

##### General purpose implementations

There is only one general-purpose implementation:
- PriorityQueue: a Queue implemented as a priority heap. Elements are ordered by natural ordering or any specific order using a Comparator.

#### Deque

A `Deque` is a linear collection of elements that supports the insertion and removal of elements at both end points, so it behaves as stack and queue. In other words, it represents a double-ended queue.

##### General purpose implementations

There are two general-purpose implementations:
- LinkedList: doubly-linked list, implementing List and Deque interfaces, providing FIFO operations.
- ArrayDeque: a Deque implemented as a resizable array, having no capacity restrictions.

### Map interface

A `Map` is an object which maps unique keys to at most one value. If you need to map a key with more than one value, use a Multimap instead. Two Map instances are equal if they represent the same key-value mappings. Although it is not a Collection per se, it contains some methods to be seen as a Collection (keySet, entrySet, values), where we can iterate to remove but not to add new elements.

##### SortedSet

A Map sorted by ascending order or keys' natural ordering or any specific one using a Comparator. It offers range-view, endpoint access and comparator access.

##### General purpose implementations

There are three general-purpose implementations:
 - HashMap
 - TreeMap
 - LinkedHashMap

, with behaviour and performance analogous to HashSet, TreeSet and LinkedHashSet, although a LinkedHashSet can also be ordered by access-order (that is, from least-recently accessed to most-recently).

<br><br><br>

So that's enough for today! In this introductory post, I have covered the most used Java Collection interfaces and general purpose implementations definitions. Apart of these ones, there are more implementations but I decided to not write about them because it would be the never ending story (and it is already perfectly explained in the Java documentation).

##### Want to know more?

- [Java tutorial] [tutorial]




[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job)

[tutorial]: https://docs.oracle.com/javase/tutorial/collections/interfaces/index.html "Open Oracle Java documentation"
[diagram]:  /img/collections_diagram.png "Java Collections Interface diagram"
