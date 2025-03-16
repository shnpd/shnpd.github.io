---
title: LeetCode1871 跳跃游戏（dp、前缀和、滑动窗口）
toc: true
date: 2021-11-04 21:28:22
tags: leetcode 动态规划 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## LeetCode1871 跳跃游戏（dp、前缀和、滑动窗口）
[题目传送门](https://leetcode-cn.com/problems/jump-game-vii/)

> 给你一个下标从 0 开始的二进制字符串 s 和两个整数 minJump 和 maxJump 。一开始，你在下标 0 处，且该位置的值一定为 '0' 。当同时满足如下条件时，你可以从下标 i 移动到下标 j 处：
i + minJump <= j <= min(i + maxJump, s.length - 1) 且
s[j] == '0'.
如果你可以到达 s 的下标 s.length - 1 处，请你返回 true ，否则返回 false 。

## 一、dp+前缀和
用dp[i]表示能否跳到i点 
dp[i]=1表示可以跳到，dp[i]=0表示不能跳到
由题意可知要想跳到i点，s[i]的值要为'0'且要能跳到i-max到i-min中的一点
我们定义left为i-max,right为i-min，即若能到达i点则区间[left,right]中至少有一个点可以到达，此时如果直接遍历此区间的话会超时，需要使用前缀和来优化。
我们定义pre[i]表示dp在区间[0,i]的和，此时要判断区间[left,right]中是否有点可以到达只需要判断pre[right]-pre[left-1]是否大于0即可。
**代码**
```cpp
class Solution
{
public:
    bool canReach(string s, int minJump, int maxJump)
    {
        vector<int> dp(s.length(), 0);
        vector<int> pre(s.length(), 0);
        dp[0] = 1;
        for (int i = 0; i < minJump; i++)
            pre[i] = 1;
        for (int i = minJump; i <= s.length() - 1; i++)
        {
            int left = i - maxJump;
            int right = i - minJump;
            if (s[i] == '0')
            {
                int total = pre[right] - (left-1>=0?pre[left-1]:0);
                if (total > 0)
                    dp[i] = 1;
            }
            pre[i] = pre[i - 1] + dp[i];
        }
        return dp[s.length() - 1];
    }
};
```

## 二、滑动窗口
如果能跳到i点，s[i]的值要为'0'且要能跳到i-max到i-min中的一点。
区间[i-max,i-min]的长度为max-min，我们可以定义变量cnt来记录次区间中能到达的点的数目。
我们从min开始遍历（因为0和min之间的点都不会到达），此时初始的区间为[min-max,min-min]此区间能到达的点只有0点，此时cnt=1
依次向后遍历，首先判断当前点能不能到，然后将区间向右移一位，如果区间右端点的下一位能到达则cnt+1，如果当前区间左端点能到达则cnt-1（因为当前左端点要被移出）。

**代码**
```cpp
class Solution
{
public:
    bool canReach(string s, int minJump, int maxJump)
    {
        vector<int> dp(s.length());
        dp[0] = 1;
        int cnt = 1;
        for (int i = minJump; i < s.length(); i++)
        {
            if (s[i] == '0' && cnt > 0)
                dp[i] = 1;
            if (dp[i - minJump + 1] == 1)
                cnt++;
            if (i - maxJump >= 0 && dp[i - maxJump] == 1)
                cnt--;
        }
        return dp[s.length() - 1];
    }
};
```

