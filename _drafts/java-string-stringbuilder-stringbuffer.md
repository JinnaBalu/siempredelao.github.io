---
layout: post
title: "String Vs StringBuilder VS StringBuffer"
author: David Aguiar
date: 2017-01-22 08:00:00
categories: Java, String, StringBuffer, StringBuilder
highlight: true
comments: true
image: /img/art-wall-brush-painting.jpg
---

Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. Introduction here. 

|              |        String        | StringBuffer | StringBuilder |
|--------------|:--------------------:|:------------:|:-------------:|
| Storage Area | Constant String Pool |     Heap     |      Heap     |
| Mutable      |          No          |      Yes     |      Yes      |
| Thread safe  |          Yes         |      Yes     |       No      |
| Performance  |         Fast         |     Slow     |      Fast     |


<br><br><br>

So that's enough for today! In this introductory post, I have covered the most used Java Collection interfaces and general purpose implementations definitions. Apart of these ones, there are more implementations but I decided to not write about them because it would be the never ending story (and it is already perfectly explained in the Java documentation).

##### Want to know more?

- [Decorator pattern (Wikipedia)] [wikipedia]




[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job)

[wikipedia]: https://en.wikipedia.org/wiki/Decorator_pattern "Open Decorator pattern in Wikipedia"
