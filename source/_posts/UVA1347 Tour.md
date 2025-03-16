---
title: UVA1347 Tour
toc: true
date: 2021-05-22 17:20:37
tags: 算法 动态规划
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

>**2021.5.22
刷题的时候突然看到手机推送，袁隆平院士逝世，心中一颤，后来得到辟谣，心情稍微放松几分，正在刷着辟谣的文章时，央视新闻发文，13点07分，袁隆平院士逝世，没过多久又看到吴孟超院士逝世的新闻，心情难以平复，特在本文的开头，向两位院士致敬。
历史浩荡，国士无双。**

## UVA1347 Tour
[题目链接](https://vjudge.net/problem/UVA-1347)

dp题，按照紫书上的分析做下来的，下面主要也是跟着紫书走一遍。

## 题目分析
“从左到右再回来”不太方便思考，可以改成：两个人同时从最左点出发，沿着两条不同的路径走，最后都走到最右点，且除了起点和终点外其余每个点恰好被一个人经过。这样，就可以用d(i,j)表示一个人走到i，另一个人走到j，还需要走的距离。
但是，如此定义，很难保证两个人不会走到同一点。
下面修改一下，d(i,j)表示从1~max(i,j)全部走过，且当前两人一个在i一个在j还需要走的距离，并且规定下一步只能走到i+1，即下一个状态为d(i+1,j)或d(i+1,i)（相当于从j点到i+1）我们规定把大的序号放在前面，否则可能会出现死循环。
边界条件是d(n-1,j)=dist(n-1,n)+dist(j,n),dist表示两点间的距离
状态转移方程为d(i,j) = min(d(i + 1, j) + dist(i, i + 1), d(i + 1, i) + dist(j, i + 1))

## AC代码

```cpp
#include<iostream>
#include<cmath>
#include<iomanip>
#include<cstring>
#include<algorithm>
using namespace std;
const int MAXN = 1000 + 10;
struct point
{
	int x, y;
	point(int a = 0, int b = 0)
	{
		x = a;
		y = b;
	}
};
point points[MAXN];
double distance(int i, int j)
{
	point a = points[i];
	point b = points[j];
	double dis = sqrt(pow((a.x - b.x), 2) + pow((a.y - b.y), 2));
	return dis;
}
int n;
double dp[MAXN][MAXN];
double d(int i, int j)
{
	if (dp[i][j] > 0)
		return dp[i][j];
	if (i == n - 1)
		return dp[i][j] = distance(i, n) + distance(j, n);
	return dp[i][j] = min(d(i + 1, j) + distance(i, i + 1), d(i + 1, i) + distance(j, i + 1));
}
int main()
{
	while (cin >> n)
	{
		memset(dp, 0, sizeof(dp));
		for (int i = 1; i <= n; i++)
		{
			cin >> points[i].x >> points[i].y;
		}
		cout << fixed << setprecision(2) << d(2, 1) + distance(1, 2) << endl;
	}
}

