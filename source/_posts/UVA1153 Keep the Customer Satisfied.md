---
title: UVA1153 Keep the Customer Satisfied
toc: true
date: 2021-05-10 18:00:27
tags: 算法 队列
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## UVA1153 Keep the Customer Satisfied
[题目链接](https://vjudge.net/problem/UVA-1153)

第一次做这种题，没有丝毫头绪，查了一下要用优先队列+贪心，在这里再记录一下。

题目大概意思就是给定n个工作的耗费时间和截止时间，求最多能完成的工作的个数。

## 解题方法
首先将输入按照截止时间从前往后排列，截止时间靠前的肯定要先完成。

然后维护一个时间和优先队列，时间就是记录当前工作的时间，优先队列存放已经完成的工作，按照所耗费的时间排序，耗时长的先出队。

**方法解释**
我们按照截止时间的升序向优先队列中push，每次向队列中插入元素时，如果插入的工作可以完成（维护时间+工作时间<=截止时间），那么直接插入即可，若当前插入的工作无法完成（维护时间+工作时间>截止时间），我们就应该贪心，可不可以用当前工作去替换一个队列中耗时更多的工作呢？以此类推，插入每个工作前都进行一次判断，最后队列的大小即为最多能完成的工作个数。

**注意**
要判断队列是否为空，可能工作本身就不符合

## AC代码

```cpp
#include<iostream>
#include<queue>
#include<algorithm>
using namespace std;
const int MAX = 8e5+10;
int T;
int N;
struct work
{
	int x, y;
	work(int a=0, int b=0)
	{
		x = a;
		y = b;
	}
};
work works[MAX];
bool cmp(work a, work b)
{
	if (a.y == b.y)
		return a.x < b.x;
	return a.y < b.y;
}
int main()
{
	int time;
	cin >> T;
	while (T--)
	{
		time = 0;
		priority_queue<int>pq;
		cin >> N;
		for (int i = 0; i < N; i++)
		{
			cin >> works[i].x >> works[i].y;
		}
		sort(works, works + N, cmp);
		
		for (int i = 0; i < N; i++)
		{
			if (time + works[i].x <= works[i].y)
			{
				pq.push(works[i].x);
				time += works[i].x;
			}
			else if (!pq.empty())
			{
				if (pq.top() >= works[i].x)
				{
					time = time - pq.top() + works[i].x;
					pq.pop();
					pq.push(works[i].x);
				}
			}
		}
		cout << pq.size() << endl;
		if (T != 0)
			cout << endl;
	}
}
```

