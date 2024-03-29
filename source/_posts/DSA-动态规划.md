---
title: 动态规划
abbrlink: a80d0031
date: 2020-10-07 15:42:23
categories:
  - Data Structure and Algorithm
---

# 最长有效括号

给定一个只包含 '(' 和 ')' 的字符串，找出最长的包含有效括号的子串的长度。
`"(()" => 2`
`")()())" => 4`
`"()(())" => 6`

```js
/**
 * 1. 定义 dp[i]:
 *    存在 0 <= x <= i, s[x]...s[i] 为以 s[i] 结尾的最长有效括号子串, dp[i] 为子串长度
 * 2. 确定状态转移方程:
 *    2.1 若 s[i] = '(', 则 dp[i] = 0
 *    2.2 若 s[i] = ')'
 *      2.2.1 若 s[i - 1] = '(', 则 dp[i] = dp[i - 2] + 2
 *      2.2.2 若 s[i - 1] = ')'
 *        2.2.2.1 若 dp[i - 1] = 0, 则 dp[i] = 0
 *        2.2.2.2 若 dp[i - 1] > 0
 *          2.2.2.2.1 若 s[i - 1 - dp[i - 1]] = '(', 则 dp[i] = dp[i - 1] + 2 + dp[i - 2 - dp[i - 1]]
 *          2.2.2.2.2 若 s[i - 1 - dp[i - 1]] = ')', 则 dp[i] = 0
 * 3. 确定初始条件:
 *    dp[0] = 0
 */
function longestValidParentheses(s) {
  let maxLen = 0;
  const dp = [0];

  for (let i = 1; i < s.length; i++) {
    if (s[i] === '(') {
      dp[i] = 0;
      continue;
    }

    /**
     * s[i] === ')'
     */
    if (s[i - 1] === '(') {
      if (i - 2 >= 0) {
        dp[i] = dp[i - 2] + 2;
      } else {
        dp[i] = 2;
      }

      maxLen = Math.max(dp[i], maxLen);
      continue;
    }

    /**
     * s[i] === ')'
     * s[i - 1] === ')'
     */
    if (dp[i - 1] === 0) {
      dp[i] = 0;
      continue;
    }

    /**
     * s[i] === ')'
     * s[i - 1] === ')'
     * dp[i - 1] > 0
     */
    if (s[i - 1 - dp[i - 1]] === '(') {
      if (i - 2 - dp[i - 1] >= 0) {
        dp[i] = dp[i - 1] + 2 + dp[i - 2 - dp[i - 1]];
      } else {
        dp[i] = dp[i - 1] + 2;
      }
      maxLen = Math.max(dp[i], maxLen);
    } else {
      dp[i] = 0;
    }
  }

  return maxLen;
}
```

# 不同路径条数

一个机器人位于一个 m x n 网格的左上角，机器人每次只能向下或者向右移动一步，机器人试图达到网格的右下角。网格中用 0 表示空位置，用 1 表示障碍物。求从左上角到右下角将会有多少条不同的路径。

```js
/**
 * !!! 需要注意横纵坐标与给定数组的对应关系
 *
 * 1. 定义dp[i][j]:
 *    从 (0, 0) 出发到 (i, j) 的路径条数
 * 2. 确定状态转移方程:
 *    2.1 若 obstacleGrid[j][i] === 1, 则 dp[i][j] = 0
 *    2.2 若 obstacleGrid[j][i] === 0, 则 dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
 * 3. 确定初始条件:
 *    dp[0][0] = 1
 */
function uniquePathsWithObstacles(obstacleGrid) {
  if (!obstacleGrid) {
    return 0;
  }

  const yLen = obstacleGrid.length;
  if (yLen <= 0) {
    return 0;
  }

  if (!obstacleGrid[0]) {
    return 0;
  }
  const xLen = obstacleGrid[0].length;
  if (xLen <= 0) {
    return 0;
  }

  const dp = [];

  /**
   * i 是横坐标
   * j 是纵坐标
   */
  for (let i = 0; i < xLen; i++) {
    dp[i] = [];

    for (let j = 0; j < yLen; j++) {
      /**
       * 初始条件
       */
      if (i === 0 && j === 0) {
        if (obstacleGrid[0][0] === 0) {
          dp[0][0] = 1;
        } else {
          dp[i][j] = 0;
        }
        continue;
      }

      dp[i][j] = 0;
      if (obstacleGrid[j][i] === 0) {
        if (i - 1 >= 0) {
          dp[i][j] += dp[i - 1][j];
        }
        if (j - 1 >= 0) {
          dp[i][j] += dp[i][j - 1];
        }
      }
    }
  }

  return dp[xLen - 1][yLen - 1];
}
```

# 编辑距离

给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数。你可以对一个单词进行三种操作：插入一个字符、删除一个字符和替换一个字符。

```js
/**
 * !!! 之所以 i, j 代表个数而不代表索引，是考虑到空字符串存在的情况，同时状态转移方程强依赖于空字符串存在的情况
 * 1. 定义dp[i][j]:
 *    word1 的前 i 个子串转换为 word2 的前 j 个子串 所需要的最少操作数
 * 2. 确定状态转移方程:
 *    2.1 若 word[i] === word[j], 则 dp[i][j] = min(dp[i][j - 1] + 1, dp[i - 1][j] + 1, dp[i - 1][j - 1])
 *    2.2 若 word[i] === word[j], 则 dp[i][j] = min(dp[i][j - 1] + 1, dp[i - 1][j] + 1, dp[i - 1][j - 1] + 1)
 * 3. 确定初始条件:
 *    dp[0][j] = j
 *    dp[i][0] = i
 */
function minDistance(word1, word2) {
  const m = word1.length;
  const n = word2.length;

  const dp = [];

  /**
   * 初始条件
   */
  for (let i = 0; i <= m; i++) {
    dp[i] = [i];
  }
  for (let j = 0; j <= n; j++) {
    dp[0][j] = [j];
  }

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      /**
       * 最后一步为插入/删除/替换操作时, 所需的步骤总数
       * 如果 word1[i - 1] === word2[j - 1], 替换操作 = 无操作
       */
      const insertActions = dp[i][j - 1] + 1;
      const deleteActions = dp[i - 1][j] + 1;
      const replaceActions =
        word1[i - 1] === word2[j - 1] ? dp[i - 1][j - 1] : dp[i - 1][j - 1] + 1;

      dp[i][j] = Math.min(insertActions, deleteActions, replaceActions);
    }
  }

  return dp[m][n];
}
```
