---
title: UVA208 Firetruck
toc: true
date: 2021-03-26 10:49:26
tags: dfs 算法 bfs
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA208 Firetruck
[题目链接](https://vjudge.net/problem/UVA-208)

刚开始做，有些细节掌握的还不太够，代码写的有些繁琐了，主要还是巩固一下思路，大家随便看看就好了。

## 问题解析
这道题理解起来不难，在这里就用紫皮书上的解释
输入一个n(n<=20)个结点的无向图以及某个结点k，按照字典序从小到大顺序输出从结点1到结点k的所有路径，要求结点不能重复经过。
提示：要事先判断结点1是否可以到达结点k，否则会超时。

## 代码分析
首先用BFS判断是否连通
然后用DFS寻找路径即可
这里要注意一些实现的小细节
1.在s输入1后记得把vis[1]标记为1
2.字典序输出要先排序
3.dfs之后不光要清楚标记，还要把s中刚刚插入的元素删除（因为这个卡了好久）
4.输出每行最后一个数字后没有空格

```cpp
#define _CRT_SECURE_NO_WARNINGS
#include<bits/stdc++.h>
using namespace std;

const int maxn = 30 + 5;
int vis[maxn];
vector<int> mp[maxn];
int mb;
int countt = 0;
int cnt = 0;
void init()
{
	memset(vis, 0, sizeof(vis));
	for (int i = 0; i < maxn; i++)
	{
		mp[i].clear();
	}
	cnt = 0;
	return;
}
void dfs(vector<int>s)
{
	int point = s[s.size()-1];
	if (point == mb)
	{
		cnt++;
		int j;
		for ( j = 0; j < s.size()-1; j++)
		{
			cout << s[j] << " ";
		}
		cout << s[j];
		cout << endl;
	}
	for (int t = 0; t < mp[point].size(); t++)
	{
		if (vis[mp[point][t]] == 1)
			continue;
		s.push_back( mp[point][t]);
		vis[mp[point][t]] = 1;
		dfs(s);
		vis[mp[point][t]] = 0;
		s.erase(s.end()-1);
	}
}
bool bfs() {
	memset(vis, 0, sizeof(vis));
	queue<int> q;
	q.push(1);
	vis[1] = 1;
	while (!q.empty()) {
		int t = q.front();
		q.pop();
		vector<int>::iterator it = mp[t].begin();
		while (it != mp[t].end())
		{
			if (!vis[*it])
			{
				vis[*it] = 1;
				if (*it == mb)
					return true;
				q.push(*it);
			}
			it++;
		}
	}
	return false;
}
int main()
{
	int a, b;
	while (cin >> mb)
	{
		vector<int>s;
		s.push_back(1);
		init();	
		while (scanf("%d%d", &a, &b) && (a != 0 || b != 0))
		{
			mp[a].push_back(b);
			mp[b].push_back(a);
		}
		for (int i = 0; i < maxn; i++)
		{
			sort(mp[i].begin(), mp[i].end());
		}
		printf("CASE %d:\n", ++countt);
		if (bfs())
		{
			memset(vis, 0, sizeof(vis));
			vis[1] = 1;
			dfs(s);
			printf("There are %d routes from the firestation to streetcorner %d.\n", cnt, mb);
		}
		else
			printf("There are 0 routes from the firestation to streetcorner %d.\n",  mb);

	}

}
```

