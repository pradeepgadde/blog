---
layout: single
title:  "The LeetCode Beginner's Guide-Day#1"
categories: Programming
tags: Python
show_date: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/python.png
author:
  name     : "Python"
  avatar   : "/assets/images/python.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---
## The LeetCode Beginner's Guide Day#1
Today, I started my programming journey with LeetCode!

Its my Day#1 and I started with Beginner's Guide.

These are the three problems I solved today.

## [2235. Add Two Integers](https://leetcode.com/problems/add-two-integers/)
Given two integers num1 and num2, return the sum of the two integers.

```py
class Solution(object):
    def sum(self, num1, num2):
        """
        :type num1: int
        :type num2: int
        :rtype: int
        """
        return int(num1)+ int(num2)
```

```sh
Accepted
Runtime: 12 ms
Case 1
Case 2
Input
num1 =
12
num2 =
5
Output
17
Expected
17
```

```sh
Input
num1 =
-10
num2 =
4
Output
-6
Expected
-6
```

## [2236. Root Equals Sum of Children](https://leetcode.com/problems/root-equals-sum-of-children/)
You are given the root of a binary tree that consists of exactly 3 nodes: the root, its left child, and its right child.

Return true if the value of the root is equal to the sum of the values of its two children, or false otherwise.


```py
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution(object):
    def checkTree(self, root):
        """
        :type root: Optional[TreeNode]
        :rtype: bool
        """
        return root.val==root.left.val+root.right.val
```

```sh
Input
root =
[10,4,6]
Output
true
Expected
true

```

```sh
Input
root =
[5,3,1]
Output
false
Expected
false
```

## [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/)

Given an array `nums`. We define a running sum of an array as `runningSum[i] = sum(nums[0]…nums[i])`.

Return the running sum of `nums`.

```py
class Solution(object):
    def runningSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        s=[]
        for i in range(len(nums)):
            s.append(sum(nums[:i+1]))
        return s
        
```

```sh
Input
nums =
[1,2,3,4]
Output
[1,3,6,10]
Expected
[1,3,6,10]
```

```sh
Input
nums =
[1,1,1,1,1]
Output
[1,2,3,4,5]
Expected
[1,2,3,4,5]
```

```sh
Input
nums =
[3,1,2,10,1]
Output
[3,4,6,16,17]
Expected
[3,4,6,16,17]

```

