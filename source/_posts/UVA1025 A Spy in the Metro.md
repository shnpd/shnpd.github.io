---
title: UVA1025 A Spy in the Metro
toc: true
date: 2021-05-21 21:43:19
tags: 算法 动态规划
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1025 A Spy in the Metro
[题目链接](https://vjudge.net/problem/UVA-1025)

刚开始接触DP题，感觉还是有一定的难度，在这里再理一遍思路。

## DP的核心就是状态和状态转移方程

首先状态的确定就是找到影响当前决策的因素，本题是当前时间和所处车站两个，所以可以用d[i][j]表示i时刻在j站最少还要等待的时间。

其次是状态转移方程，取决于决策的形式，本题主要有以下三种决策：
1.在当前车站等待一分钟。
2.搭乘向右开的车。
3.搭乘向左开的车。
找出决策后即可列出相对应的状态转移方程。
1.dp[i][j]=min(dp[i][j],dp[i+1][j]+1)
2.dp[i][j]=min(dp[i][j],dp[i+t[j][j+1])
3.dp[i][j]=min(dp[i][j],dp[i+t[j-1]][j-1])

## 下一步要找出边界条件和计算顺序（递推法）
这两步就要结合题意来做具体的分析
本题的边界条件为T时刻到达车站n即dp[T][n]=0
计算顺序不难判断应该从后往前，这里有一个小技巧，可以根据状态转移方程，不难看出T小的值要根据T大的求出，所以应该先求T大的值。

## AC代码
注意数组范围尽量开大一些，防止越界（Runtime error）

```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
#define INF 0x3f3f3f3f
using namespace std;
const int MAXN = 50+10;
const int MAXM = 50+10;
const int MAXT = 300;
int n, T, t[MAXN], M1, tr[MAXM], M2, tl[MAXM];
bool has_train[MAXT][MAXN][2];
int dp[MAXT][MAXN];
bool init()
{
	cin >> n;
	if (n == 0)
		return false;
	int time;
	memset(dp, INF, sizeof(dp));
	memset(has_train, false, sizeof(has_train));
	memset(t, 0, sizeof(t));
	cin >> T;
	for (int i = 1; i < n; i++)
		cin >> t[i];
	cin >> M1;
	time = 0;
	for (int i = 0; i < M1; i++)
	{
		cin >> tr[i];
		time = tr[i];
		has_train[time][1][0] = true;
		for (int i = 1; i < n; i++)
		{
			time += t[i];
			has_train[time][i + 1][0] = true;
		}
	}
	cin >> M2;
	for (int i = 0; i < M2; i++)
	{
		cin >> tl[i];
		time = tl[i];
		has_train[time][n][1] = true;
		for (int i = 1; i < n; i++)
		{
			time += t[n - i];
			has_train[time][n - i][1] = true;
		}
	}
	return true;
}

void solve()
{
	for (int i = 1; i < n; i++)
		dp[T][i] = INF;
	dp[T][n] = 0;
	for (int i = T-1; i >= 0; i--)
	{
		for (int j = 1; j <= n; j++)
		{
			dp[i][j] = min(dp[i][j], dp[i + 1][j] + 1);
			if (j < n && has_train[i][j][0] && i + t[j] <= T)
				dp[i][j] = min(dp[i][j], dp[i + t[j]][j + 1]);
			if (j > 1 && has_train[i][j][1] && i + t[j - 1] <= T)
				dp[i][j] = min(dp[i][j], dp[i + t[j - 1]][j - 1]);
		}
	}
}

int main()
{
	int cnt = 0;
	while (init())
	{
		solve();
		cout << "Case Number " << ++cnt << ": ";
		if (dp[0][1] >= INF)
			cout << "impossible\n";
		else
			cout << dp[0][1] << "\n";
	}
	return 0;
}

