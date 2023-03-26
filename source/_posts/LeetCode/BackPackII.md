---
title: BackPackII
date: 2023-03-26 19:42:53
categories:
    - LeetCode
tags: 
    - Dynamic Programming
---


```java
package leetcode.problems.medium;

import util.PrintUtil;

public class BackPackII {
    /**
     * 状态 dp(i, j): 前 i个物品放入大小为 j的背包中所获得的最大价值
     * weight(i): 新增物品的大小
     * value(i): 新增物品的价格
     * 递推关系:
     * 不放: dp(i, j) = dp(i-1, j) 表示不把第i个物品放入背包中，所以它的价值就是前i-1个物品放入大小为j的背包的最大价值
     * 放: dp(i-1, j - weight[i]) + value[i] 表示把第i个物品放入背包中，价值增加value[i],但是需要腾出j - weight[i]的大小放
     * 状态转移方程: dp(i,j) = max{ dp(i-1, j), dp(i-1, j - weight[i]) + value[i] }
     * 初始状态：dp(i, 0) = dp(0, j) = 0   第0行和第0列都为0，表示没有装物品时的价值都为0
     * 返回值：dp(i, j)
     * @param m 大小为 m 的背包
     * @param A 每个物品的大小
     * @param V 每个物品的价值
     * @return 最多能装入背包的总价值是多大
     */
    public int backPackII(int m, int[] A, int[] V) {
        int itemNum = A.length;
        int weight[] = A;
        int value[] = V;
        if (itemNum == 0 || m == 0) return 0;
        // dp[最多取物品数量个][最大取背包大小]
        int[][] dp = new int[itemNum][m + 1];
        // 初始化
        for (int i = 0; i < dp.length; i++) { // 可以省略
            dp[i][0] = 0;
        }
        for (int j = 0; j < m + 1; j++) {
            if (j < weight[0]) {
                dp[0][j] = 0;
            } else {
                dp[0][j] = value[0];
            }
        }

        for (int i = 1; i < itemNum; i++) { // 遍历背包
            for (int j = 0; j <= m; j++) { // 遍历重量
                if (weight[i] > j) {
                    // 新增物品质量大于当前背包，放不下
                    dp[i][j] = dp[i - 1][j];
                } else {
                    // 放得下，如果放入新物品要计算出背包剩余大小，看下剩余背包大小最多能装多少然后加上新增物品价格，和不放入新物品背包最大价格对比，取最大。最大价格都在上一层
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
                }
            }
        }
        PrintUtil.printSubList(dp);
        return dp[itemNum-1][m];
    }

    /**
     * 一维数组（滚动数组）实现
     * 状态 F(i, j): 前 i个物品放入大小为 j的背包中所获得的最大价值
     * A(i): 新增物品的大小
     * V(i): 新增物品的价格
     * 状态转移方程: dp(i,j) = max{ dp(i-1, j), dp(i-1, j - A[i-1]) + V[i-1] }
     * dp[i] = 前 i个物品放入大小为 j的背包中所获得的最大价值
     * dp[j] = max(dp[j], dp[j - A[i-1]] + V[i-1]);
     *
     * @param m 大小为 m 的背包
     * @param A 每个物品的大小
     * @param V 每个物品的价值
     * @return 最多能装入背包的总价值是多大
     */
    public int backPackII1(int m, int[] A, int[] V) {
        int itemNum = A.length;
        int weight[] = A;
        int value[] = V;
        if (itemNum == 0 || m == 0) return 0;
        // dp[最大取背包大小]
        int[] dp = new int[m + 1];
        // 初始化
        for (int j = 0; j < m + 1; j++) {
            if (j < weight[0]) {
                dp[j] = 0;
            } else {
                dp[j] = value[0];
            }
        }
        for (int i = 1; i < itemNum; i++) { // 遍历背包
            for (int j = m; j >= weight[i]; j--) { // 遍历重量
                dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i]);
            }
        }
        PrintUtil.printSubList(dp);
        return dp[m];
    }
}
```


