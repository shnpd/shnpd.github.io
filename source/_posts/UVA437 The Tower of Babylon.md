---
title: UVA437 The Tower of Babylon
toc: true
date: 2021-05-22 11:56:48
tags: 算法 动态规划
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA437 The Tower of Babylon
[题目链接](https://vjudge.net/problem/UVA-437)

动态规划

## 题目
有n(n≤30)种立方体，每种都有无穷多个。要求选一些立方体摞成一根尽量高的柱子(可以自行选择哪一条边作为高)，使得每个立方体的底面长宽分别严格小于它下方立方体的底面长宽。

## 分析
题目中有两句话比较重要，一是每种立方体都有无穷多个，二是可以自行选择哪一条边作为高，所以当输入为n种立方体时，可供我们选择的立方体个数一共是3*n个，每输入一种立方体，对应的就有三种立方体供我们选择（分别以输入的三个数作为高）。
然后根据立方体之间能否摞在一起的关系可以建立有向无环图（DAG），利用邻接矩阵存储。
最后直接dp，用dp[i]存储从第i个立方体开始最高的高度，最后遍历取得最大值即可。

## AC代码

```cpp
#include<iostream>
#include<cstring>
#include<algorithm>
using namespace std;
const int MAX = 1000;
int n;
struct rec
{
	int a, b, h;
	rec(int x = 0, int y = 0, int z = 0)
	{
		a = x;
		b = y;
		h = z;
	}
};
rec recs[200];
int t1, t2, t3;
int G[200][200];
bool judge(rec x, rec y)//x能否放在y上面
{
	if ((x.a < y.a && x.b < y.b) || (x.a < y.b && x.b < y.a))
		return true;
	return false;
}
void graph()//如果i能放在j上面则G[i][j]=1
{
	memset(G, 0, sizeof(G));
	for (int i = 0; i < 3 * n; i++)
	{
		for (int j = 0; j < 3 * n; j++)
		{
			if (judge(recs[i], recs[j]))
				G[i][j] = 1;
		}
	}
}
int dp[200];
int d(int i)
{
	if (dp[i] > 0)
		return dp[i];
	dp[i] = recs[i].h;
	for (int j = 0; j < 3 * n; j++)
	{
		if (G[j][i])
			dp[i] = max(dp[i], d(j) + recs[i].h);
	}
	return dp[i];
}
int main()
{
	int kase = 0;
	while (cin >> n && n)
	{
		for (int i = 0; i < n; i++)
		{
			cin >> t1 >> t2 >> t3;
			recs[3 * i].a = t1;
			recs[3 * i].b = t2;
			recs[3 * i].h = t3;

			recs[3 * i + 1].a = t2;
			recs[3 * i + 1].b = t3;
			recs[3 * i + 1].h = t1;

			recs[3 * i + 2].a = t3;
			recs[3 * i + 2].b = t1;
			recs[3 * i + 2].h = t2;
		}
		graph();
		memset(dp, 0, sizeof(dp));
		int ans = 0;
		for (int t = 0; t < 3 * n; t++)
		{
			ans = max(ans, d(t));
		}
		printf("Case %d: maximum height = %d\n", ++kase, ans);
	}
}

