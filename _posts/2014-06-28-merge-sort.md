---
title: Merge Sort
author: buraktas
layout: post
permalink: /merge-sort/
math: true
dsq_thread_id:
  - 2802018068
categories:
  - algorithms
  - sorting
tags:
  - algorithms
  - java
  - merge-sort
  - sorting
comments: true
---
Merge sort is another comparison based sort algorithm. It closely follows *divide-and-conquer* paradigm, and provides \\( O(n&#8203;lgn) \\) run time complexity in worst and average cases, however, \\( O(n) \\) in space complexity. Merge sort can be used for sorting huge data sets that can not fit into memory. It is also a stable sort which preserves the order of equal elements.

<!--more-->

<h2> Algorithm </h2>

Merge sort follows a divide-and-conquer algorithm by the following steps;

<div>
  <ul>
    <li>
      <b>Divide :</b> Divide the n-element sequence into two subsequences of n/2 elements each
    </li>
    <li>
      <b>Conquer :</b> Sort the two subsequences recursively using merge sort.
    </li>
    <li>
      <b>Combine :</b> Merge the two sorted subsequences to produce the sorted answer.
    </li>
  </ul>
</div>

<h3> Complexity Analysis </h3>

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
        \(O(n&#8203;lgn)\)
      </td>

      <td>
        \(O(n&#8203;lgn)\)
      </td>

      <td>
        \(O(n&#8203;lgn)\)
      </td>

      <td>
        \(O(n) auxiliary\)
      </td>
    </tr>
</table>

<h2> Code </h2>

<em>mergeSort</em> procedure divides input into two equal parts by recursion until input size becomes 1. Then, <em>merge</em> procedure sorts and combine divided parts. The code of mergeSort procedure is as following ;

<pre><code class="language-java">public void mergeSort(int[] numbers, int left, int right) {
	if (left &lt; right) {
		int mid = (left + right) / 2;

		// divide array into two equal parts
		mergeSort(numbers, left, mid);
		mergeSort(numbers, mid + 1, right);

		// merge part
		merge(numbers, left, mid, right);
	}
}</code>
</pre>

![]({{ site.url }}/assets/img/2014/06/merge_sort.png)

<em>merge</em> procedure is the part where subarrays are combined and sorted. Thus, <em>merge</em> procedure on an n-element subarray takes time \\( O(n) \\). Code of <em>merge</em> procedure is as follows;

<pre><code class="language-java">private void merge(int[] numbers, int left, int mid, int right) {
	int[] leftArray = new int[mid - left + 1];
	int[] rightArray = new int[right - mid];

	// copy values into created arrays
	System.arraycopy(numbers, left, leftArray, 0, leftArray.length);
	System.arraycopy(numbers, mid + 1, rightArray, 0, rightArray.length);

	int leftIndex = 0;
	int rightIndex = 0;
	int k = left;

	while (leftIndex &lt; leftArray.length && rightIndex &lt; rightArray.length) {

		if (leftArray[leftIndex] &lt;= rightArray[rightIndex]) {
			numbers[k] = leftArray[leftIndex];
			leftIndex++;
		} else {
			numbers[k] = rightArray[rightIndex];
			rightIndex++;
		}
		k++;
	}

	// copy rest of the left array
	while (leftIndex &lt; leftArray.length) {
		numbers[k] = leftArray[leftIndex];
		leftIndex++;
		k++;
	}

	// copy rest of the right array
	while (rightIndex &lt; rightArray.length) {
		numbers[k] = rightArray[rightIndex];
		rightIndex++;
		k++;
	}
}</code>
</pre>

![combine_step]({{ site.url }}/assets/img/2014/06/merge_sort_combine_and_sort_step.png)
