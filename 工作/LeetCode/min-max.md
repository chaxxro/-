
# min-max

## lc486、lc877

```cpp
/*
1. 记忆化递归
定义一个函数 f，返回从 [l, r] 区间中取一个左值/右值后能获取的最大分数差
f(l, r) = max(num[l] - f(l + 1, r), num[r] - f(l, r - 1))

2. dp
*/
```

[分析](https://www.youtube.com/watch?v=g5wLHFTodm0)

[题目](https://leetcode.com/problems/predict-the-winner/description/)

[分析](https://www.youtube.com/watch?v=xJ1Rc30Pyes)

[题目](https://leetcode.com/problems/stone-game/description/)

## lc1140

```cpp
/*
dp[i][j] 表示从 i 开始，M = j 能获得的最大分数差
dp[i][j] = max(A[i] + ... + A[x] - dp[x + 1][max(x, M)])
*/
```

[分析](https://www.youtube.com/watch?v=e_FrC5xavwI)

[题目](https://leetcode.com/problems/stone-game-ii/description/)

[分析](https://www.youtube.com/watch?v=uzfsrChj8dM)

[题目](https://leetcode.com/problems/stone-game-iii/description/)

[题目](https://leetcode.com/problems/stone-game-iv/description/)

## lc2017

[题目](https://leetcode.com/problems/grid-game/description/?envType=daily-question&envId=2025-01-21)
