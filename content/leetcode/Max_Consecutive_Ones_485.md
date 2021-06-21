---
title: "485. Max Consecutive Ones"
date: 2021-06-20T20:45:22-04:00
draft: false
tags: [Algorithms, Arrays, Programming]
---

Here is my thought process and solutions for Leetcode problem [485. Max Consecutive Ones](https://leetcode.com/problems/max-consecutive-ones). 

> Given a binary array nums, return the maximum number of consecutive 1's in the array.

You need to do three things to solve this problem:
* Determine if the current index is a `1` or a `0`
* Find the index ranges with consecutive `1`s
* Compare these index ranges to find the range with the maximum length
* Return the length of that range

I solved this problem in: **Python3**, **Javascript**, **mySQL**, and **Java**:

{{< max_consecutive_ones >}}