---
title: UVA11134 Fabled Rooks
toc: true
date: 2021-04-02 15:37:37
tags: 算法 贪心算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA11134 Fabled Rooks

[题目链接](https://vjudge.net/problem/UVA-11134)

## 问题解析
在n行n列的棋盘上放n个车，第i个车在给定的矩形Ri之内，且任意两个车不能相互攻击，即任意两个车不在同一行或同一列。求每个车的位置坐标。

首先，一个车所在的行不会影响它所在的列，可以分别求一个车的横坐标和纵坐标，将一个二维的问题转化为一个一维的问题。

现在，我们面临的问题是如何将这n个点分给n个车。
以横坐标xi为例，每个车的x都被限定在了一个区间[xli,xri]之内，不难看出应该应用贪心算法求解。


这里我最开始做的是按照xli从小到大排序，如果相同就按照xri从小到大排序，然后从每个区间的最左端开始遍历，找到比前一个点的x大的就取出。但会遗漏一种情况，即当输入数据为
3
1 1 3 3
1 1 3 3
2 2 2 2
（这里大家可以自己代入看一下，因为是错误方法就不多解释了）

下面是正确的解题思路，因为我们是从左往右取点，如果一个点被多个区间包含，不难判断，我们应该取右端点最小的区间，因为右端点越大，它所能取的值就越多，所以对区间的排序，应该是按照右端点从小到大，如果相同就按左端点从小到大。取点时按照区间从左往右，注意判断当前所取的点之前没有取过。


## 代码
以下代码可以AC，但时间复杂度较高代码比较繁琐，大家看看思路即可。

```cpp
#include<bits/stdc++.h>
using namespace std;

const int maxn = 5000 + 10;
typedef pair<int, int> pos;
int N;
struct point 
{
	int num;
	pos resx;
	pos resy;
	int x, y;
};
point points[maxn];
bool cmp1(point a, point b)
{
	if (a.resx.second == b.resx.second)
		return a.resx.first < b.resx.first;
	return a.resx.second < b.resx.second;
}
bool cmp2(point a, point b)
{
	if (a.resy.second == b.resy.second)
		return a.resy.first< b.resy.first;
	return a.resy.second < b.resy.second;
}
bool sortx()
{
	sort(points, points + N, cmp1);
	points[0].x = points[0].resx.first;
	int i, j;
	bool flag = false;
	for (i = 1; i < N; i++)
	{
		for ( j = points[i].resx.first; j <= points[i].resx.second; j++)
		{
			flag = false;
			for (int t = 0; t < i; t++)
			{
				if (j == points[t].x)
				{
					flag = true;
					break;
				}
			}
			if (!flag)
			{
				points[i].x = j;
				break;
			}
		}
		if (j > points[i].resx.second)
			return false;
	}
	return true;
}
bool sorty()
{
	sort(points, points + N, cmp2);
	points[0].y = points[0].resy.first;
	int i, j;
	bool flag = false;
	for (i = 1; i < N; i++)
	{
		for (j = points[i].resy.first; j <= points[i].resy.second; j++)
		{
			flag = false;
			for (int t = 0; t < i ; t++)
			{
				if (j == points[t].y)
				{
					flag = true;
					break;
				}
			}
			if (!flag)
			{
				points[i].y = j;
				break;
			}
		}
		if (j > points[i].resy.second)
			return false;
	}
	return true;
}
void output()
{
	for (int i = 0; i < N; i++)
	{
		for (int j = 0; j < N; j++)
		{
			if (points[j].num == i)
			{
				cout << points[j].x << " " <<points[j].y<< endl;
				break;
			}
		}
	}
}
int main()
{
	int x1, y1, x2, y2;
	while (cin >> N && N != 0)
	{
		for (int i = 0; i < N; i++)
		{
			cin >> x1 >> y1 >> x2 >> y2;
			points[i].num = i;
			points[i].resx = make_pair(x1, x2);
			points[i].resy = make_pair(y1, y2);
		}
		if (!sortx())
			cout << "IMPOSSIBLE"<<endl;
		else
		{
			if (!sorty())
				cout << "IMPOSSIBLE"<<endl;
			else
				output();
		}
	}
}
```

