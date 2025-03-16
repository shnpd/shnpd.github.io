---
title: UVA1617 Laptop
toc: true
date: 2021-05-17 21:12:25
tags: 算法 贪心算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1617 Laptop
[题目链接](https://vjudge.net/problem/UVA-1617)

## 题意
这里引用紫皮书上的解释

给定n条长度为1的线段，确定它们的起点，使得第i条线段在[ri,di]之间。输入保证ri≤rj，当且仅当di≤dj，且保证有解。输出空隙数目的最小值。

对这个题目有一点疑问，给定的样例中有[1,3]和[0,3]两个区间，假定[1,3]为第i条线段[0,3]为第j条线段，di≤dj，但是ri>dj，不满足题意所述，猜想可能是题目有误（VJ上的题目也是此条件）。

## 思路
一道蛮简单的贪心题，首先按照左端点小的在前排序，若左端点相等，则对右端点小的在前排序，然后尽量靠右选点，维护最右端的点ri。

不难判断，当前所选线段与前一个有空隙的情况只有当前线段的左端点大于ri。此时必定有空隙，只需对当前尽量靠右选点即可。

当没有空隙时，首先判断是否可以选ri右边的点，此时需满足当前线段的右端点大于ri。如果右端点等于ri则将ri前移一个，将ri位置空出来给当前线段使用（不必考虑ri是否可以前移，因为题目中提到必定有解），此时最右端的点仍是之前ri的值，如果右端点小于ri，则当前线段选的点在ri左边，ri仍不变。

## AC代码

```cpp
#include<iostream>
#include<algorithm>
using namespace std;
struct line
{
	int l, r;
	line(int a = 0, int b = 0)
	{
		l = a;
		r = b;
	}
	bool operator <(const line a)
	{
		if (l == a.l)
			return r < a.r;
		return l < a.l;
	}
};
const int MAXN = 1e5 + 10;
int T, N;
line lines[MAXN];
int main()
{
	int ri;
	cin >> T;
	while (T--)
	{
		cin >> N;
		for (int i = 0; i < N; i++)
		{
			cin >> lines[i].l >> lines[i].r;
		}
		sort(lines, lines + N);
		ri = lines[0].r;
		int cnt = 0;
		for (int i = 1; i < N; i++)
		{
			if (lines[i].l > ri)
			{
				cnt++;
				ri = lines[i].r;
				continue;
			}
			if (lines[i].r > ri)
			{
				ri++;
				continue;
			}
		}
		cout << cnt << endl;
	}
}
```

