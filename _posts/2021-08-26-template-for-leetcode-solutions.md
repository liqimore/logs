---
layout: post
title: "Templates for leetcode problem solutions"
description: "Templates for leetcode problem solutions"
date: 2021-08-26
tags: [leetcode, job]
categories: [job]
comments: true
---

* table of contents
{:toc .toc}

## Problem description

Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

You can return the answer in any order.

Example 1:

> Input: nums = [2,7,11,15], target = 9
>> Output: [0,1]

Because nums[0] + nums[1] == 9, we return [0, 1].

<https://leetcode-cn.com/problems/two-sum>

## Analysis

### Enumerate

Go through the list and find the one matches `target -x`.

### Hashtable

Search in hashtable for matches of `target -x` then insert `x` itself.

## Solutions

### Java

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> hashtable = new HashMap<Integer, Integer>();
        for (int i = 0; i < nums.length; ++i) {
            if (hashtable.containsKey(target - nums[i])) {
                return new int[]{hashtable.get(target - nums[i]), i};
            }
            hashtable.put(nums[i], i);
        }
        return new int[0];
    }
}
```

Time complexity: O(N), where NN is the number of elements in the array. For each element x, we can find target-x O(1)O(1).

Space complexity: O(N), where NN is the number of elements in the array. Mainly the overhead of the hash table.

### Golang

```golang
func twoSum(nums []int, target int) []int {
    for i, x := range nums {
        for j := i + 1; j < len(nums); j++ {
            if x+nums[j] == target {
                return []int{i, j}
            }
        }
    }
    return nil
}
```

Time complexity: O(N^2), where NN is the number of elements in the array. In the worst case, any two numbers in the array must be matched once.

Space complexity: O(1).

## Reference

<https://leetcode-cn.com/problems/two-sum/solution/liang-shu-zhi-he-by-leetcode-solution/>
