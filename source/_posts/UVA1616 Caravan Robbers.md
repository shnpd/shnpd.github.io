---
title: UVA1616 Caravan Robbers
toc: true
date: 2021-05-12 21:33:41
tags: 算法 二分法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1616 Caravan Robbers
[题目链接](https://vjudge.net/problem/UVA-1616)

二分+小数转分数

题意：给定n个区间，把它们变成等长的不想交的区间，求区间的最大长度。
注意本题精度要求较高，注意浮点数的比较方式。
## 思路
**1.二分**
通过对题目的分析，我们不难发现，所有小于最大长度的数都满足不相交，所有大于最大长度的数都会相交，满足单调性，可以通过二分来求解最大长度。

通过二分来求区间的最大长度，首先选定左端点l=0，右端点r为n各区间中最靠右的点，然后求mid=(r+l)/2,判断此时的mid是否满足各区间不想交，若不满足，则mid应该大于最大长度，向前二分。反之，则mid应该小于等于最大长度，向后二分，最后左端点即为最大长度。

**2.小数转分数**
一共有n个区间，开始时这n个区间的长度都是整数，我们假设最极端的情况这n个区间都在[0,1]中，此时最大长度为1/n，所以分母一共只有1~n这n种情况(这一段是从网上找的，我也没弄明白为什么只有n种情况，有明白的大佬可以在评论区留言讨论一下)。然后枚举分母即可。

## AC代码

```cpp
#include<iostream>
#include<cstring>
#include<cmath>
#include<algorithm>
using namespace std;
const int MAXN = 1e5 + 10;
const double eps = 1e-11;
struct line
{
	int l, r;
	line(int a = 0, int b = 0)
	{
		l = a;
		r = b;
	}
	bool operator < (const line x)
	{
		if (l == x.l)
			return r < x.r;
		return l < x.l;
	}
};
int n;
line lines[MAXN];
bool judge(long double x)
{
	long double ans = 0;//区间起点
	for (int i = 0; i < n; i++)
	{
		ans = max(ans, (long double)lines[i].l);
		ans += x;//区间终点
		if (ans - lines[i].r>eps)
			return false;
	}
	return true;
}
int main()
{
	int maxr = 0;
	while (cin >> n && n)
	{
		for (int i = 0; i < n; i++)
		{
			cin >> lines[i].l >> lines[i].r;
			maxr = max(maxr, lines[i].r);
		}
		sort(lines, lines + n);
		long double ll = 0, rr = maxr;
		while (rr - ll > eps)
		{
			long double mid = (rr + ll) / 2;
			if (judge(mid))//如果符合mid可能小于等于最大长度，向后找
				ll = mid;
			else//不符合mid大于最大长度,向前找
				rr = mid;
		}
		int t, k;
		for (k = 1; k <= n; k++)
		{
			t = ll * k + 0.5;
			if (fabs((long double)t / k - ll) < eps)
				break;
		}
		cout << t << '/' << k << endl;
	}
}

