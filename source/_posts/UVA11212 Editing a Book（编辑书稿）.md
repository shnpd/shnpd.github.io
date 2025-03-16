---
title: UVA11212 Editing a Book（编辑书稿）
toc: true
date: 2021-03-22 20:37:55
tags: 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA11212 Editing a Book
[题目链接](https://vjudge.net/problem/UVA-11212)

这道题有一定的难度，参考了一些大佬的代码，在这里写一篇题解巩固一下。

## 问题分析
首先，本题采用IDA*算法求解，参考紫皮书上的解释，这道题启发函数的求解是解题的一个关键。

其次是三个加速的策略：
1.每次只剪切一段连续的数字。
2.假设剪切片段的第一个数字为a，最后一个数字为b，这个片段要么粘贴到a-1下一个位置，要么粘贴到b+1的前一个位置。
3.已经排好序的数字不要分开剪切，始终看做一个整体。
（以上解释纯属搬书，下面结合代码具体分析）

最后，启发函数的求解，设最多剪切maxd次，d是已经剪切的次数，后继不正确的数字还有h个，不难发现，每次剪切最多可以改变3个数字的直接后继。
如下图，把2移到3之后，只有1后继变为3,3后继变为2,2后继变为4.
![](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/cfd0ea27ce425e65d3befcb44ae7d286_1740931367206.png)
由此可得当h>3*(maxd-d)时可以剪枝。

## 代码

```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<iostream>
#include<cstdio>
#include<vector>
#include<algorithm>
using namespace std;
int geth(vector<int>& nums)
{
	int cnt = 0;
	for (int i = 1; i < nums.size(); ++i)
	{
		if (nums[i - 1] + 1 != nums[i])
			++cnt;
	}
	return cnt;
}
bool dfs(int d, vector<int>& nums, int maxd)
{
	int i = 0, cnt = geth(nums);
	//剪枝
	if (cnt == 0)
		return true;
	if (cnt > 3 * (maxd - d))
		return false;
	for (i = 0; i < nums.size();)
	{
		int j = i + 1;
		while (j < nums.size() && nums[j - 1] + 1 == nums[j])
			++j;

		for (; j <= nums.size(); ++j)
		{
			vector<int>tmp = nums;
			vector<int>in(nums.begin() + i, nums.begin() + j);
			tmp.erase(tmp.begin() + i, tmp.begin() + j);
			vector<int>lhs = tmp, rhs = tmp;
			auto lp = find(lhs.begin(), lhs.end(), in.front() - 1);
			auto rp = find(rhs.begin(), rhs.end(), in.back() + 1);
			lhs.insert(lp == lhs.end() ? lhs.begin() : next(lp),      in.begin(), in.end());
			rhs.insert(rp, in.begin(), in.end());
			if (dfs(d + 1, lhs, maxd) || dfs(d + 1, rhs, maxd))
				return true;
		}
		i++;
		while (i < nums.size() && nums[i - 1] + i == nums[i])
			++i;
	}
	return false;
}
int main()
{
	int N, kase = 0;
	while (scanf("%d", &N) != EOF && N)
	{
		vector<int>nums(N + 1, 0);
		for (int i = 1; i <= N; i++)
			scanf("%d", &nums[i]);
		for (int maxd = 0;; ++maxd)
		{
			if (dfs(0, nums, maxd))
			{
				printf("Case %d: %d\n", ++kase, maxd);
				break;
			}
		}
	}
	return 0;
}
```

代码是借鉴的大佬的，如果大家有什么不明白的地方，可以在评论区交流一下。
