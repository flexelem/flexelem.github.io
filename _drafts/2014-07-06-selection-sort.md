---
title: Selection Sort
author: buraktas
layout: post
permalink: /selection-sort/
dsq_thread_id:
  - 2822289059
categories:
  - Algorithms
  - Sorting
tags:
  - algorithms
  - java
  - selection-sort
  - sorting
---
Selection sort is an in-place comparison sort algorithm which has \(O(n^2)\) running time complexity in both best and worst case. Even though its logic is similar to insertion sort, it&#8217;s a very inefficient sorting algorithm. List is both divided into two parts as sorted and unsorted where initially sorted part is empty. Every turn, algorithm finds the minimum (or maximum) key in unsorted sublist then swaps it with the leftmost element of unsorted part. Due to the reason which elements in any place could get swapped selection sort is not stable.

### Algorithm 

<div class="bullet list">
  <ul>
    <li>
      Find minimum in unsorted list.
    </li>
    <li>
      Swap minimum with left most element in unsorted list.
    </li>
    <li>
      Shift boundaries of unsorted list to right by one.
    </li>
  </ul>
</div>

[<img src="http://www.buraktas.com/wp-content/uploads/2014/07/selection_sort.png" alt="selection_sort" width="1233" height="938" class="aligncenter size-full wp-image-445" />][1]

### Complexity Analysis 

Due to the fact that selection sort is not adaptive ( number of cycles doesn&#8217;t change due to given order) it takes \(O(n^2)\) for every case.

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
      Best case
    </th>
    
    <th>
      Worst case
    </th>
    
    <th>
      Average case
    </th>
    
    <th>
      Worst case
    </th>
  </tr>
  
  <tr>
    <td>
      \(O(n^{2})\)
    </td>
    
    <td>
      \(O(n^{2})\)
    </td>
    
    <td>
      \(O(n^{2})\)
    </td>
    
    <td>
      \(O(1) auxiliary\)
    </td>
  </tr>
</table>

### Code 

<pre class="lang:java decode:true " >public class SelectionSort {

    public int[] selectionSort(int[] numbers) {

        for (int i = 0; i &lt; numbers.length - 1; i++) {

            int minIndex = i;
            for (int j = i + 1; j &lt; numbers.length; j++) {
                if (numbers[j] &lt; numbers[minIndex]) {
                    minIndex = j;
                }
            }

            if (minIndex != i) {
                swap(i, minIndex, numbers);
            }
        }
        return numbers;
    }

    private void swap(int i, int j, int[] numbers) {
        int temp = numbers[i];
        numbers[i] = numbers[j];
        numbers[j] = temp;
    }
}</pre>

 [1]: http://www.buraktas.com/wp-content/uploads/2014/07/selection_sort.png