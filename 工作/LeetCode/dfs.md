
# dfs

## lc329


[题目](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)

[分析](https://www.bilibili.com/video/BV1mW411d7q8?from=search&seid=5099018830887943293)

## lc332

```cpp
/*
有向图遍历边
1. 每次都贪心地选择最小字节序的结点
2. 遇到死路时，而没有把所有的边都走过一遍的话，就说明这种走法不满足条件，需要沿着树根向上找到最近的一个有其他路可以走的节点把新的路走一遍

题目保证一定存在一条满足要求的条件路径，那么一条这样的死路，一定会相对的在这个节点上存在另一条路，这条路存在一个回到该节点的环

先把这个环走过之后再去走这条死路，就可以保证把以这个节点的所有点都走到
*/
```

[题目](https://leetcode.com/problems/reconstruct-itinerary/)

[分析](https://www.youtube.com/watch?v=4udFSOWQpdg)

## lc399

[题目](https://leetcode.com/problems/evaluate-division/description/?envType=study-plan-v2&envId=top-interview-150)

## lc865、lc1123

[题目](https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/description/)

[题目](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/)

## Lc1079

[题目](https://leetcode.com/problems/letter-tile-possibilities/description/?envType=daily-question&envId=2025-02-17)

## lc1593

[题目](https://leetcode.com/problems/split-a-string-into-the-max-number-of-unique-substrings/description/)

## lc2115

[题目](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/description/?envType=daily-question&envId=2025-03-21)

## Lc2467

[题目](https://leetcode.com/problems/most-profitable-path-in-a-tree/?envType=daily-question&envId=2025-02-24)

[分析](https://www.bilibili.com/video/BV1qe4y1G7jy/?vd_source=85004908aeb75ae8d616c254af2684fe)

## lc3373

[题目](https://leetcode.com/problems/maximize-the-number-of-target-nodes-after-connecting-trees-ii/description/?envType=daily-question&envId=2025-05-29)
