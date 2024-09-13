---
title: Binary Search
author: buraktas
layout: post
permalink: /binary-search/
math: true
dsq_thread_id:
  - 2770434619
categories:
  - algorithms
  - searching
tags:
  - algorithms
  - binary-search
  - java
comments: true
---
Binary search is a searching algorithm which works on a sorted (ascending or descending) array. In each turn, the given key is compared with the key at the middle of the array, if it is equal to it then its index is returned. Otherwise, if the given key is less than key at the middle then binary search continues its operation on left sub-array \\( ( A[0..Mid-1] ) \\), and similarly on right sub-array \\( ( A[mid+1..N-1] ) \\) if given key greater than it. If the given key cannot be found then an indication of not found is returned.

<!--more-->

Finding an item takes logarithmic time in binary search. Thus, it works blazingly fast on huge input sets. For example, only 16 iteration is needed to search for an **int** value on a 2GB array. The step by step work flow is as follows;

<div>
  <ol>
    <li>
      Check if <b>key</b> = A[mid] .
    </li>
    <li>
      If <b>key</b> > A[mid] , then continue to search from right half, and return to step 1.
    </li>
    <li>
      If <b>key</b> < A[mid] , then continue to search from left half, and return to step 1.
    </li>
  </ol>
</div>

<h2> Complexity Analysis </h2>

<table class="TFtable">
  <tr>
    <th colspan="3">
      Time
    </th>

    <th>
      Space
    </th>
  </tr>

  <tr>
    <th>
      Best Case
    </th>

    <th>
      Average Case
    </th>

    <th>
      Worst Case
    </th>

    <th>
      Worst Case
    </th>
  </tr>

  <tr>
    <td>
      \(O(1)\)
    </td>

    <td>
      \(O(lgn)\)
    </td>

    <td>
      \(O(lgn)\)
    </td>

    <td>
      \(O(1) auxiliary\)
    </td>
  </tr>
</table>

An illustration of searching a key 22 is shown below;

![binary_search]({{ site.url }}/assets/img/2014/06/binary_search1.png)

<h2> Code </h2>

```java
public int binarySearch(int[] numbers, int left, int right, int key) {
    if (left <= right) { // recurse until length of array is 1
      int mid = (left + right) / 2;

      if (key == numbers[mid]) {
        return mid;
      }

      if (key > numbers[mid]) { // continue from right sub-array
        return binarySearch(numbers, mid + 1, right, key);
      } else { // continue from left sub-array
        return binarySearch(numbers, 0, mid - 1, key);
      }
    }

    return -1;
}
```
