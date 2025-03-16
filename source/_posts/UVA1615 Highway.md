---
title: UVA1615 Highway
toc: true
date: 2021-05-09 12:33:39
tags: 贪心算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1615 Highway
[题目链接](https://vjudge.net/problem/UVA-1615)

这道题不算难，就是一道贪心题，下面介绍两种方法。

**第一种方法**
主要的思路就是在x轴上向右选取距离当前点的长度为D的点，即最靠右的点，这样选可以尽可能的满足后面的点。

选点前先将每个点从左向右排序，横坐标相同则从上往下排，这里注意要把纵坐标小的放在后面，因为纵坐标小所选取的点更靠右，如果把纵坐标小的放在前面那么下一个纵坐标大的点距离这个点一定大于D不满足，会多选一个点。（这里表达不太清楚，可以自己画个图看一下）

**第二种方法**
主要的思路就是区间选点，每个点在x轴上都有一个选点的区间，我们把这N个区间选取出来，然后选取最少的点保证每个区间都有点即可。

## AC代码
**方法一**

```cpp
#include<iostream>
#include<algorithm>
#include<cmath>
using namespace std;
const int MAXN = 1e5;
int L, D;
struct point
{
	int x;
	int y;
	point(int a = 0, int b = 0)
	{
		x = a;
		y = b;
	}
};
point points[MAXN];

double getp(int a, int b)//在x轴上向右找到距离(a,b)小于等于L的最远的点
{
	double x = sqrt(pow(D, 2) - pow(b, 2)) + a;//距离等于L的点
	if (x >L)
		return L;
	return x;
}
bool judge(point p, int x)//判断p点距离(x,0)是否小于D
{
	int a = p.x;
	int b = p.y;
	double dis =pow(x - a, 2) + pow(b, 2);
	if (dis <= pow(D, 2))
		return true;
	return false;
}
bool cmp(point a, point b)
{
	if (a.x == b.x)
		return a.y > b.y;
	return a.x < b.x;
}
int main()
{
	while (cin >> L)
	{
		cin >> D;
		int N;
		cin >> N;
		int cnt = 0;
		double x;
		for (int i = 0; i < N; i++)
		{
			cin >> points[i].x >> points[i].y;
		}
		sort(points, points + N, cmp);
		cnt++;
		x = getp(points[0].x, points[0].y);
		for (int i = 0; i < N; i++)
		{
			if (judge(points[i], x))
				continue;
			else
			{
				x = getp(points[i].x, points[i].y);
				cnt++;
			}
		}
		cout << cnt << endl;
	}
	return 0;
}
```
**方法二**
```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
#include<cmath>
using namespace std;
const int MAXN = 1e5;
int L, D;
struct qu
{
	double l;
	double r;
	qu(double x=0, double y=0)
	{
		l = x;
		r = y;
	}
};
qu qujian[MAXN];
bool cmp(qu a, qu b)
{
	if (a.l == b.l)
		return a.r < b.r;
	return a.l < b.l;
}
int xuandian(int n)
{
	int cnt = 1;
	double r = qujian[0].r;
	for (int i = 0; i < n; i++)
	{
		if (qujian[i].l <= r)
			continue;
		else
		{
			r = qujian[i].r;
			cnt++;
		}
	}
	return cnt;
}
int main()
{
	while (cin >> L)
	{
		memset(qujian, 0, sizeof(qujian));
		int a, b, N;
		cin >> D >> N;
		for (int i = 0; i < N; i++)
		{
			cin >> a >> b;
			double r = a + sqrt(pow(D, 2) - pow(b, 2));//右端点
			double l = a - sqrt(pow(D, 2) - pow(b, 2));//左端点
			qujian[i].l = l;
			qujian[i].r = r;
		}
		sort(qujian, qujian + N, cmp);
		cout<<xuandian(N)<<endl;
	}
	return 0;
}

```

