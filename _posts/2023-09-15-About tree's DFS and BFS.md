---
title: About tree's DFS and BFS
date: 2023-09-15 11:11:00 -0700
categories: [algorithm]
tags: 
Pin:
---

## What is the DFS

- *Depth First Search* (`DFS`)

  In this strategy, we adopt the `depth` as the priority, so that one would start from a root and reach all the way down to a certain leaf, and then back to the root to reach another branch.

  The DFS strategy can further be distinguished as `preorder`, `inorder`, and `postorder` depending on the relative order among the root node, left node, and right node.

## What is the BFS

- *Breadth First Search* (`BFS`)

  We scan through the tree level by level, following the order of height, from top to bottom. The nodes on higher levels would be visited before the ones with lower levels.

In the following figure, the nodes are enumerated in the order you visit them, please follow `1-2-3-4-5` to compare different strategies.

![img](https://raw.githubusercontent.com/wabol/pic/master/2024/02/upgit_20240207_1707324838.png)

test

![image-20240207090747016](https://raw.githubusercontent.com/wabol/pic/master/2024/02/upgit_20240207_1707325667.png)