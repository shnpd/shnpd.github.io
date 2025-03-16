---
title: UVA1601 The Morning after Halloween
toc: true
date: 2021-03-22 20:56:43
tags: 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1601 The Morning after Halloween
[题目链接](https://vjudge.net/problem/UVA-1601)

做这道题的时候看到一个写的很好的代码，在这里保存下来，以便以后学习。

## 题目分析
这道题和普通的bfs有所不同，解题方法也有些差别，主要是这里有三个移动的“小鬼”，每个小鬼有五种移动状态（上下左右和不动），最主要的是可以看出在地图上大部分的点都是障碍物，所以可以把所有的空格都提出来建立一张图，而不必每次判断五种方法是否合法，以此来优化算法。

## 代码解析
本题代码中用到的变量较多，为便于理解，先把主要变量的含义解释一下。

x[i] :第i个空格的横坐标
y[i] :第i个空格的纵坐标
id[i][j] :坐标为(i,j)的空格的编号
s[i]:初始小鬼所在空格的编号
t[i]:小鬼所要到达的目标位置的空格编号
deg[i]:在第i个空格所能走的步数
G[i][j]:在第i个空格所能到达的第j个位置的空格编号

**注意事项：**
main函数中最后两个if语句的作用是当小鬼的数量小于三个时，默认多出的小鬼初始位置即为目标位置，且他们只能原地不动。
ID函数的作用是将三个变量合成一个变量一次性存到队列中。
```cpp
#define _CRT_SECURE_NO_WARNINGS
#include <iostream>
#include <cstdio>
#include <queue>
#include <cstring>
using namespace std;

int w, h, n, s[3], t[3];
char dataset[20][20];
int G[200][5], vis[200][200][200], dist[200][200][200];
int deg[200];

int dx[] = { 0,-1,1,0,0 };
int dy[] = { 0,0,0,-1,1 };

inline int ID(int a, int b, int c)
{
	return(a << 16) | (b << 8) | c;
}
inline bool conflict(int a, int b, int a2, int b2)
{
	return((a2 == b2) || (a == b2 && b == a2));
}
int bfs()
{
	queue<int>q;
	q.push(ID(s[0], s[1], s[2]));
	dist[s[0]][s[1]][s[2]] = 0;
	while (!q.empty())
	{
		int u = q.front();
		q.pop();
		int a = (u >> 16) & 0xff;
		int b = (u >> 8) & 0xff;
		int c = u & 0xff;
		if (a == t[0] && b == t[1] && c == t[2])
			return dist[a][b][c];
		for (int i = 0; i < deg[a]; i++)
		{
			int a2 = G[a][i];
			for (int j = 0; j < deg[b]; j++)
			{
				int b2 = G[b][j];
				if (conflict(a, b, a2, b2))
					continue;
				for (int k = 0; k < deg[c]; k++)
				{
					int c2 = G[c][k];
					if (conflict(a, c, a2, c2) || conflict(b, c, b2, c2))
						continue;
					if (dist[a2][b2][c2] == -1)
					{
						dist[a2][b2][c2] = dist[a][b][c] + 1;
						q.push(ID(a2, b2, c2));
					}
				}
			}
		}
	}
	return -1;
}
int main()
{
	while (~scanf("%d%d%d\n", &w, &h, &n) && n)
	{
		for (int i = 0; i < h; i++)
			fgets(dataset[i], 20, stdin);
		int cnt = 0, x[200], y[200], id[20][20];
		for (int i = 0; i < h; i++)
		{
			for (int j = 0; j < w; j++)
			{
				if (dataset[i][j] != '#')
				{
					x[cnt] = i;//第cnt个空格的横坐标
					y[cnt] = j;//第cnt个空格的纵坐标
					id[i][j] = cnt;//坐标为(i,j)的空格的编号
					if (islower(dataset[i][j]))
						s[dataset[i][j] - 'a'] = cnt;//初始小鬼位置
					else if (isupper(dataset[i][j]))
						t[dataset[i][j] - 'A'] = cnt;//目标小鬼位置
					cnt++;
				}
			}
		}
		for (int i = 0; i < cnt; i++)
		{
			deg[i] = 0;//在第i个空格所能走的步数
			for (int j = 0; j < 5; j++)
			{
				int nx = x[i] + dx[j];
				int ny = y[i] + dy[j];
				if (dataset[nx][ny] != '#')
					G[i][deg[i]++] = id[nx][ny];//第i个空格所能到达的位置编号
			}
		}
		if (n <= 2)
		{
			deg[cnt] = 1;
			G[cnt][0] = cnt;
			s[2] = t[2] = cnt++;
		}
		if (n <= 1)
		{
			deg[cnt] = 1;
			G[cnt][0] = cnt;
			s[1] = t[1] = cnt++;
		}
		memset(dist, -1, sizeof(dist));
		printf("%d\n", bfs());
	}
	return 0;
}
```

