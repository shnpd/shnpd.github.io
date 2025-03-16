---
title: LeetCode486.预测赢家（递归+动态规划+博弈）
toc: true
date: 2021-11-02 10:44:48
tags: leetcode 动态规划 算法 递归算法 博弈论
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

# LeetCode486.预测赢家（递归+动态规划+博弈）
[题目传送门](https://leetcode-cn.com/problems/predict-the-winner/)

## 题目解析

> 给你一个整数数组 nums 。玩家 1 和玩家 2 基于这个数组设计了一个游戏。
玩家 1 和玩家 2 轮流进行自己的回合，玩家 1 先手。开始时，两个玩家的初始分值都是 0 。每一回合，玩家从数组的任意一端取一个数字（即，nums[0] 或 nums[nums.length - 1]），取到的数字将会从数组中移除（数组长度减 1 ）。玩家选中的数字将会加到他的得分上。当数组中没有剩余数字可取时，游戏结束。
如果玩家 1 能成为赢家，返回 true 。如果两个玩家得分相等，同样认为玩家 1 是游戏的赢家，也返回 true 。你可以假设每个玩家的玩法都会使他的分数最大化。

两个玩家轮流从一个数组中取数字，每次只能取两端的数字，如果先手取得的数字总和大于等于后手，则返回true，否则返回false，每个玩家的玩法都是最优的。

## 递归
这里我们采用净胜分来判断，如果是玩家1得分，则净胜分加上得分，如果是玩家2得分，则净胜分减去得分，最终如果净胜分大于等于0，则玩家1获胜。
我们用一个solve(vector<int>nums,int start,int end)函数来求当前玩家的最大得分，当前玩家有两种选择：
一、选择首部的数字，此时他的最大得分为：**nums[start]-solve(vector&lt;int&gt;nums,start+1,end)**
其中solve(vector&lt;int&gt;nums,start+1,end)为下一个玩家的得分，当前玩家的净胜分应该减去下一个玩家的得分。
二、选择尾部的数字，此时他的最大得分为：**nums[end]-solve(vector&lt;int&gt;nums,start,end-1)**
得到当前玩家的两种可能得分后，取最大值即可。
**代码**

```cpp
class Solution
{
public:
    bool PredictTheWinner(vector<int> &nums)
    {
        return solve(nums, 0, nums.size()-1) >= 0;
    }
    int solve(vector<int> &nums, int start, int end)
    {
        if (start == end)
        {
            return  nums[start];
        }
            int stasco = nums[start] - solve(nums, start + 1, end);
            int endsco = nums[end] - solve(nums, start, end - 1);
            return max(stasco, endsco);      
    }
};
```

## 动态规划
首先我们假设dp[i][j]为nums[i]到nums[j]中玩家的最大得分
通过上面递归的解法我们可以得到状态转移方程为：**dp[i][j]=max(nums[i]-dp[i+1][j],nums[j]-dp[i][j-1])**
当i>j时：dp[i][j]无意义
当i=j时：dp[i][j]=nums[i]
当i<j时：dp[i][j]=max(nums[i]-dp[i+1][j],nums[j]-dp[i][j-1])
要求dp[i][j]我们首先要求dp[i+1][j]以及dp[i][j-1]，因此我们需要从下往上，从左往右求解。
**代码**

```cpp
class Solution
{
public:
    bool PredictTheWinner(vector<int> &nums)
    {
        int n = nums.size();
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int i = 0; i < nums.size(); i++)
            dp[i][i] = nums[i];
        for(int i=n-2;i>=0;i--)
        {
            for(int j=i+1;j<=n-1;j++)
            {
                dp[i][j]=max(nums[i]-dp[i+1][j],nums[j]-dp[i][j-1]);
            }
        }
        return dp[0][n-1]>=0;
    }
};
```

## 博弈（数组个数为偶数，先手必胜）
这里代码有一个优化的点就是当数组个数为偶数时，先手必胜。
假设当前数组为{1,7,5,26,8,4}
我们将数组数组分为奇数列{1,5,8}和偶数列{7,26,4}
比较两个数列之和显然偶数列大，那么只要我们先手一直选择偶数列的数，就可以获胜。因为当我们先手选了偶数列时，此时数列的两端都只剩下奇数列的数，对方只能选择奇数列，对方选完之后我们继续选择偶数列的数，最终我们可以得到全部偶数列的数，进而获得胜利。
同理，当奇数列大时，我们选择奇数列即可。
当两数列之和相同时，我们可以任意选择，因为平局也认为是我们获胜。


