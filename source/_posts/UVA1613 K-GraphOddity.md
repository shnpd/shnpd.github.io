---
title: UVA1613 K-GraphOddity
toc: true
date: 2021-05-08 21:04:11
tags: dfs 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1613 K-GraphOddity
[题目传送门](https://vjudge.net/problem/UVA-1613)

刚看第一眼一点思路都没有，后面看了大佬的题解发现这道题其实是一道水题，用到的方法就是DFS遍历图。~~我是废物~~ 

题目意思很简单，就不分析了，下面直接说方法。

首先求出k，然后dfs遍历一遍图，给每个点分一个数字即可。这里注意每次dfs判断的是当前结点的所有相邻接点，“分配颜色”时从1到k遍历一遍找到合适的即可。

## 代码

```cpp
#include<iostream>
#include<vector>
#include<cstring>
using namespace std;
vector<int> gp[10010];//保存边
int ans[10010];//保存每个结点的染色
int k;
bool judge(int c, int p)//判断点p能不能染c色
{
	for (int i = 0; i < gp[p].size(); i++)
	{
		if (ans[gp[p][i]] == c)
			return false;
	}
	return true;
}
void dfs(int x)
{
	for (int i = 0; i < gp[x].size(); i++)
	{
		if (ans[gp[x][i]] != 0)
			continue;
		for (int j = 1; j <= k; j++)
		{
			if (judge(j, gp[x][i]))
			{
				ans[gp[x][i]] = j;
				break;
			}
		}
		dfs(gp[x][i]);
	}
}
int main()
{
	int a, b;
	int n, m;
	while (cin >> n >> m)
	{
		for (int i = 0; i < m; i++)
		{
			cin >> a >> b;
			gp[a].push_back(b);
			gp[b].push_back(a);
		}
		k = gp[1].size();
		for (int i = 1; i <= n; i++)//求k
		{
			if (gp[i].size() > k)
				k = gp[i].size();
		}
		if (k % 2 == 0)
			k++;
		cout << k << endl;
		ans[1] = 1;
		dfs(1);
		for (int i = 1; i <= n; i++)
		{
			cout << ans[i] << endl;
		}
		cout << endl;
		for (int i = 1; i <= n; i++)
		{
			gp[i].clear();
		}
		memset(ans, 0,sizeof(ans));
	}
}```


