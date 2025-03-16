---
title: UVA1614 Hell on the Markets
toc: true
date: 2021-05-08 22:50:11
tags: 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1614 Hell on the Markets
[题目传送门](https://vjudge.net/problem/UVA-1614)

这道题主要考察的数学推理能力，一点思路都没有，大佬的分析都看了好久。

这道题主要用到的是数学归纳法，用sum[i]表示前i项的和，从1到sum[i]之间的任意一个数，都可以用a[1]到a[i]中的若干个数表示出来。

具体证明方法以及选数的方法可以看下面这篇文章
[https://blog.csdn.net/weixin_30820151/article/details/101380416](https://blog.csdn.net/weixin_30820151/article/details/101380416?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162047936216780255212994%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=162047936216780255212994&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_v2~rank_v29-1-101380416.pc_search_result_cache&utm_term=uva1614)

## 代码

```cpp
#include<iostream>
#include<cstring>
using namespace std;
const int MAXN = 1e5 + 10;
int A[MAXN];
int fh[MAXN];
int main()
{
	int n;
	while (cin >> n)
	{
		long long  sum = 0;
		memset(fh, 0, sizeof(fh));
		memset(A, 0, sizeof(A));
		for (int i = 0; i < n; i++)
		{
			cin >> A[i];
			sum += A[i];
		}
		if (sum & 1)
		{
			cout << "No" << endl;
			continue;
		}
		cout << "Yes" << endl;
		sum /= 2;
		for (int j =n-1; j >= 0; j--)
		{
			if (sum - A[j] >= 0)
			{
				sum -= A[j];
				fh[j] = 1;
			}
			else
			{
				fh[j] = -1;
			}
		}
		for (int i = 0; i < n; i++)
		{
			cout << fh[i] << " ";
		}
		cout << endl;
	}
}
```

