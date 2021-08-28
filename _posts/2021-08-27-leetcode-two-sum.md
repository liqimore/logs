---
layout: post
title: "1. Two Sum"
description: "1. Two Sum in leetcode"
date: 2021-08-27
tags: [leetcode, job, TwoSum]
categories: [job]
comments: true
---

* table of contents
{:toc .toc}

## Problem

<https://leetcode-cn.com/problems/two-sum/>

## Solutions

### Enumerate

Go through `nums[]` array and set `i = len`, `j = i + 1`. Then loop every possible cases by using 2 for loop.

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int n = nums.length;

        for (int i = 0; i < n; i++) {

            for (int j = i + 1; j < n ; j++) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        
        return new int[]{};

    }
}
```

```golang
func twoSum(nums []int, target int) []int {
	n := len(nums)
	for i := 0; i < n; i++ {
		for j := i + 1; j < n; j++ {
			if nums[i] + nums[j] == target {
				return []int{i,j}
			}
		}
	}
	return []int{};
}
```

Time complexity: O(N^2). N is the length of num array. If possible, loop every elements in the arrray.

Space Complexity: O(1).

### Hashtable

In order to get indices i and j, we need target = nums[i] + nums[j], which is the same as target - nums[i] = nums[j].

So, if current nums[i] matches to target - nums[i], we find the indexes (hashtale(target-nums[i]), i). It basicly using
a hashtable to represent the other value(i or j) from the enumerate version.

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {

        // key = elements in nums, value = index
        Map<Integer, Integer> hashMap = new HashMap<>(nums.length);

        for (int i = 0; i < nums.length; i++) {
            if (hashMap.containsKey(target - nums[i])) {
                return new int[]{hashMap.get(target - nums[i]), i};
            }
            hashMap.put(nums[i], i);
        }
        return new int[]{};

    }
}
```

```golang
func twoSum(nums []int, target int) []int {

	hashmap := make(map[int]int)

	for i := 0; i < len(nums); i++ {
		if v, ok := hashmap[target - nums[i]]; ok {
			return []int{v, i}
		}
		hashmap[nums[i]] = i
	}
	return nil
}
```

Time complexity: O(N). N is the length of num array. If possible, loop every elements in the arrray **ONCE**.

Space Complexity: O(N), we saved elements dynamically in a hashtable.

## Reference

<https://leetcode-cn.com/problems/two-sum/solution/liang-shu-zhi-he-by-leetcode-solution/>
