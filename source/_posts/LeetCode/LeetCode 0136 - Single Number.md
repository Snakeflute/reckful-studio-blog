---
title: LeetCode 0136 - Single Number
date: 2018-05-28 14:40:00
categories: LeetCode
---
# Single Number

<!--more-->

## Desicription

Given a **non-empty** array of integers, every element appears twice except for one. Find that single one.

**Note**:

Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

**Example** 1:

```
Input: [2,2,1]
Output: 1
```

**Example** 2:

```
Input: [4,1,2,1,2]
Output: 4
```

## Solution

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for(auto it : nums)
            res ^= it;
        return res;
    }
};
```