---
layout: post
title: "Practice programming with code katas"
excerpt: Kata is an exercise where the novice repeatedly tries to emulate a master. In karate, these kata are a sequence of basic moves, strung together in a way that makes sense. PragDave proposes the idea of doing the same with programming.
modified: 2016-10-20 14:05:22 +0200
categories: articles
tags: [kata, javascript, binary search, binary tree, algorithm, data structure]
image:
  feature: 2016-10-20-practice-programming-with-code-katas/cover.jpg
  credit: Fervent Jan
  creditlink: https://unsplash.com/search/martial-art?photo=kKLEgX-Z_O4
comments: true
share: true
published: true
aging: false
---

Dave Thomas (a.k.a. [@PragDave](https://twitter.com/pragdave "@PragDave Twitter profile")) proposes the idea of code katas.

> [Kata](https://en.wikipedia.org/wiki/Kata "Kata Wikipedia page") (Japanese for form or pattern) are an exercise where the novice repeatedly tries to emulate a master. In karate, these kata are a sequence of basic moves (kicks, blocks, punches, and so on), strung together in a way that makes sense. You’ll never be attacked in such a way that you could repeat a kata to defend yourself: that isn’t the idea. Instead the idea is to practice the feel and to internalize the moves. (Interestingly, kata are not just used in the martial arts. Calligraphers also learn using kata, copying their masters’ brush strokes.)

The idea is that just like in martial arts, we should practice programming just for the sake of practicing. During each exercise you need to look for feedback so you could make improvements the next time.

PragDave has a [website](http://codekata.com/ "CodeKata") on which he lists a number of example exercises. I'm going to be looking at the one that involves [implementing a binary search algorithm 5 days in a row](http://codekata.com/kata/kata02-karate-chop/ "Karate Chop Kata") . The caveat is that every day it should be implemented differently. At the same time you should observe what kind of common errors you encounter and how do you improve over the course of 5 days.

## Binary search in short

[Binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm "Binary search Wikipedia page") is a search algorithm which finds the target element in a **sorted array** by comparing the target value to the middle element of the array. If they are unequal, half of the array is discarded and the search continues on the half where the element may still be found. Since the array is sorted, the algorithm can always determine which part of the array to eliminate. For example, if the target element is greater than the middle element, then the target element cannot be to the left of the middle element.

## Specification

I needed to write a function that accepts an integer and a sorted array. It should return `-1` if the integer is not present in the array. Otherwise it should return the index of the integer in the array.

## Day 1

The first day was easy because I had at least two techniques in mind—loops and recursion. I decided to implement a binary search algorithm with a while loop. The program would take the middle element of the array and at each iteration choose if the target element is to the left or to the right of the selected element, essentially halving the search space.

Each iteration after the first would only consider the sub-array that was either to the left or to the right of the previous middle element. This loop continues until the selected middle element of a sub-array is the target element or the sub-array is empty, in which case `-1` is returned.

{% highlight javascript %}
function chop(int, array) {
  var l = 0;
  var r = array.length - 1;

  while (l <= r) {
    var m = Math.floor((l+r)/2);
    if (array[m] < int) {
      l = m + 1;
    }
    else if (array[m] > int) {
      r = m - 1;
    }
    else {
      return m;
    }
  }

  return -1;
}
{% endhighlight %}

Beware of infinite loops. It's easy to use a wrong boolean expression for the while loop. Additionally off by one errors are common I guess.

## Day 2

Since I already implemented a solution with a while loop, the next logical step was to use recursion. Initially I implemented a recursive function which accepted slices of arrays. For example, when the target element was to the left of the middle point, a sub-array from index 0 to middle point was passed to the recursive function. Apparently I could not get this to work.
It returned an index from the sub-array but the goal is to get an index from the original array.

I knew there was a solution for this but it was late in the evening and I decided to fall back to a backup solution. I changed the function signature to accept the left and right indexes of an array. They represent the sub-array where the target element might be found.

{% highlight javascript %}
function chop(int, array) {
  var innerChop = function(left, right) {
    if (left > right) return -1;
    var m = Math.floor((left + right)/2);

    if (array[m] < int) {
      return innerChop(m + 1, right);
    }
    else if (array[m] > int) {
      return innerChop(left, m - 1);
    }
    else {
      return m;
    }
  }
  return innerChop(0, array.length - 1);
}
{% endhighlight %}

## Day 3

I was starting to run out of ideas. Then I remembered that on day 2 I failed to implement my initial idea. Instead of passing left and right indexes to the recursive function, I wanted to pass a sub-array. But the recursive algorithm returned an incorrect result. It returned an index from the sub-array but it should return an index from the original array.

When the target element is to the right of the middle element, I needed to add the middle element index + 1 to the result of the recursive call. This assured that I would get a result that corresponds to an index in the original array. There's one exception though. When the target element does not exist the function has to return `-1`. So there has to be an explicit check for a missing result and `-1` needs to be propagated up the call stack.

{% highlight javascript %}
function chop(int, array) {
  if (array.length == 0) return -1;

  var m = Math.floor((array.length -1)/2);
  if (array[m] == int) {
    return m;
  }
  else if (array[m] < int) {
    var result = chop(int, array.slice(m + 1));
    return result == -1 ? -1 : 1 + m + result;
  }
  else {
    return chop(int, array.slice(0, m));
  }
}
{% endhighlight %}

## Day 4

It took me a while to come up with the next solution. I have already used recursion and a while loop. What else is there to use?

Since kata is used to practice and the solutions don't have to be *production ready* I decided to think outside the box. Instead of working on the array, it could be used to build a binary search tree. Later I can use recursion to traverse the tree and find the correct answer. Yes, I'm making the solution more complex than it needs to be, but I could not come up with anything better.

Nonetheless, I continued and built a binary search tree from a sorted array. The algorithm is quite simple actually.

1. Get the Middle of the array and make it root.
2. Recursively do same for left half and right half.
  * Get the middle of left half and make it left child of the root created in step 1.
  * Get the middle of right half and make it right child of the root created in step 1.

{% highlight javascript %}
function Node(value, index, left, right) {
  this.value = value;
  this.index = index;
  this.left = left;
  this.right = right;
}

function buildTree(array, index) {
  if (array.length === 0) return null;
  var mid = Math.floor((array.length - 1) / 2);

  var leftTree = buildTree(array.slice(0, mid), index);
  var rightTree = buildTree(array.slice(mid + 1), index + mid + 1);

  return new Node(array[mid], index + mid, leftTree, rightTree);
}

function search(int, node) {
  if (node == null) {
    return -1;
  }
  else if (node.value === int) {
    return node.index;
  }
  else if (int < node.value) {
    return search(int, node.left);
  }
  else {
    return search(int, node.right);
  }
}

function chop(int, array) {
  var root = buildTree(array, 0);
  return search(int, root);
}
{% endhighlight %}

The trick here is to return the original index from the array. This means I had to store the index in a tree node. The solution is kind of similar to the solution on day 3 where I had to keep track of the original index. For all nodes to the right or the parent, I had to add the parent's node index + 1.

## Day 5

I already went the length to create a binary search tree and traverse the tree recursively to find the correct solution. What if I did that without recursion. Actually it's not that difficult at all. Since the tree is already sorted, there's no need to backtrack up the tree. The algorithm has to choose to go left or right. If there's no where to go, this means the target element is not in the tree.

By the way, I'm using the same tree building code that was displayed on day 4.

{% highlight javascript %}
function search(int, node) {
  while(node != null) {
    if (int === node.value) {
      return node.index;
    }
    else if (int > node.value) {
      node = node.right;
    }
    else {
      node = node.left;
    }
  }

  return -1;
}

function chop(int, array) {
  var root = bst.buildTree(array, 0);
  return search(int, root);
}
{% endhighlight %}

## Summary

Day 1 to 3 where relatively easy. But after that, coming up with additional solutions took some time. I guess it's hard to think outside the box if you're used to thinking inside it. When writing these kind of algorithms you can encounter off-by-one errors and infinite loops, so be aware.

It is not necessary to arrive at a correct solution. In most cases I think there is no correct answer. What's important is what you learn along the way. After all, the goal is to practice.

*All solutions can be found on [Github](https://github.com/indrekots/katas/tree/master/kata02-karate-chop "Binary chop repository").*
