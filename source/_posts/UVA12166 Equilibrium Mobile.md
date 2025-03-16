---
title: UVA12166 Equilibrium Mobile
toc: true
date: 2021-03-13 16:32:27
tags: 算法 dfs
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

[VJ传送门](https://vjudge.net/problem/UVA-12166)

一道思维题，刚开始看的时候没什么思路，在博客园上参考了大佬的解析，在这里总结一下。

## 一、分析
这道题要求让天平平衡所需要的最小改动次数，至少有一个不变，我们可以先选定一个不变的基准，然后改变其他的秤砣，得到以此为基准的天平的总重量，如果以深度为d重量为w的秤砣为基准，那么整个天平的重量就是w * pow(2, d)，即w << d。

当然，可能会有一些秤砣算出的以各自为基准的天平总重量相同，设天平总重量为sumw，那么这些秤砣的数量就表示了如果使天平的总重量为sumw需要使多少个秤砣保持不变。

以题目所给[[3,7],6]为例，如果将3看做基准，则将7改为3即可，天平总重量为12，再将6看做基准，将7改为3即可，天平总重量为12,即有两个秤砣作为基准得到的天平总重量相同为12，则如果使天平的总重量为12需要使两个秤砣保持不变。 

基于以上算法，求出所有可能的sumw值以及其对应的秤砣数量，然后在这些sumw值中找到相对应的最大的秤砣数量maxn，那么sum - maxn即为所求。
## 二、代码

```cpp
#include<iostream>
#include<string>
#include<map>

using namespace std;

map<long long, int>mp;
int sum = 0;//叶子个数
string s;
void dfs(int depth, int a, int b)//depth:深度，a：起点，b：终点
{
	if (s[a] == '[')
	{
		int p = 0;
		for (int i = a + 1; i != b; i++)
		{
			if (s[i] == '[')
				p++;
			if (s[i] == ']')
				p--;
			if (p == 0 && s[i] == ',')
			{
				dfs(depth + 1, a + 1, i - 1);//搜索左子树
				dfs(depth + 1, i + 1, b - 1);//搜索右子树
			}
		}
	}
	else
	{
		long long w = 0;
		//这里是将string类型的数字转化为long long类型
		for (int i = a; i <= b; i++)
		{
			w = w * 10 + s[i] - '0';
		}
		sum++;
		mp[w * (1 << depth)]++;
	}
}
int main()
{
	int T;
	cin >> T;
	while (T--)
	{
		sum = 0;
		mp.clear();
		cin >> s;
		dfs(0, 0, s.length() - 1);
		int maxn = 0;
		for (map<long long, int>::iterator it = mp.begin(); it != mp.end(); it++)
		{
			maxn = max(maxn, it->second);
		}
		cout << sum - maxn << endl;
	}
}

