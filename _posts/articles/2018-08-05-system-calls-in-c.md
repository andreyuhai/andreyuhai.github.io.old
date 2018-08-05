---
layout: post
title: "System Calls in C"
excerpt: System calls and how to make them. System and exec functions...
modified:
categories: articles
tags: [C, system calls]
image:
  feature: 2018-08-05-system-calls-in-c/pavan-trikutam-1660-unsplah.jpeg 
  credit: Pavan Tritukam
  creditlink: https://unsplash.com/photos/71CjSSB83Wo 
comments: true
share: true
published: true
aging: true
---

A system call is a request for service that a program makes of the kernel. An example to that would be the `system()` function.

`int system(const char* command);`

**Returns** -1 on error or the return status of the command otherwise.

But since this function would run any command given to it as a string you would probably like to use something specific like `exec()` functions.

### 1. `exec()` Functions
Using `exec()` functions you can remove the ambiguity of which program to run by telling the operating system precisely which program to run.

Basically there are two types of `exec()` functions. _List functions_ and _array functions_. `exec()` functions are in the library `unistd.h`

**NOTE:** Since `exec()` functions replace the current process. After you call the `exec()` function somewhere in your code, the code below won't be executed because of the aforementioned reason. You may want to search for the `fork()` function for that which I am not going to talk about in this article.

#### 1.1 List Functions: `execl()`, `execlp()`, `execle()`


