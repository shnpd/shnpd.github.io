---
title: KMP算法（严蔚敏数据结构第二版）
toc: true
date: 2021-04-09 12:23:05
tags: 字符串
categories: 数据结构
---

​​点击阅读更多查看文章内容<!--more-->

KMP算法之前看过一次，看了好久才看明白，今天又学的时候发现啥也不会了，又看了好久，在这里整理一下思路，方便以后复习。

## 算法介绍
在我们常规的模式匹配算法中，每当匹配失败时，模式串都从第一个字符开始重新比较，KMP算法的改进在于：当匹配中出现字符不相等时，主串指针不回溯，模式串指针根据部分匹配的结果，尽可能的向右“滑动”一段距离，从而减少匹配次数。
kmp算法可以在O(n+m)的时间数量级上完成串的模式匹配操作。
算法匹配过程如下：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/99b55dc2bd4e0e3a34caeae1902907f5_1740931367206.jpeg)
这里可以看到第三次匹配时，模式串没有从头开始，而是直接比较了b，这是KMP算法的难点之一，**即当匹配过程中产生失配时，主串中的第i个字符应该与模式中哪个字符比较呢？**

这里我们以模式串s="abaab"为例，假设在比较s[3]处失配了，那么这时再比较可以从s[1]处开始，因为我们知道s[0]~s[2]匹配成功，即s[2]对应的主串位置应为'a'，s[0]=s[2]所以s[0]不需要再次匹配直接放到之前s[2]所在的位置即可，不难发现，这里移动的位置应该与模式串的前缀字符串和后缀字符串有关，这就是下面要讲的next数组。

## next数组
next[j]=k：表示当模式中的第j个字符与主串中相应字符失配时，在模式串中需重新和主串中该字符进行比较的字符位置。

注：下标从1开始


|j|1 |2| 3 |4| 5| 6| 7| 8  |
|--|--|--|--|--|--|--|--|--|
|模式串| a| b| a |a |b |c| a| c| 
next[j] |0 |1 |1 |2 |2 |3 |1 |2|


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/28b08512ff1470113bdff09750cf30b9_1740931367206.jpeg)

## KMP代码

```cpp
int Index_KMP(string S, string T, int pos)
{
	//利用模式串T的next函数求T在主串S中第pos个字符之后的位置
	//其中T非空，1<=pos<=S.length
	int i = pos; 
	int j = 1;
	while (i <= S.length() && j <= T.length())
	{
		if (j == 0 || S[i] == T[j])//匹配成功，继续比较后续字符
		{
			++i;
			++j;
		}
		else
			j = next[j];//模式串右移
	}
	if (j > T.length())
		return i - T.length();//匹配成功
	else
		return 0;//匹配失败
}
```

## 求解next函数

><font color="red">**先放一个b站的视频，使用图解法解释代码，非常形象，建议大家直接看视频就好了**</font>
>[https://www.bilibili.com/video/BV16X4y137qw/](https://www.bilibili.com/video/BV16X4y137qw/)


next[j]=k,表明在模式串中
"t[1]t[2]...t[k-1]"="t[j-k+1]t[j-k+2]...t[j-1]"(假设下标从1开始)
此时求解next[j+1]有以下两种情况
1.t[k]=t[j]
则在模式串中有"t[1]t[2]...t[k-1]t[k]"="t[j-k+1]t[j-k+2]...t[j-1]t[j]"
此时next[j+1]=next[j]+1
2.t[k]≠t[j]
此时可以把求next函数值的问题看成是一个模式匹配的问题，整个模式串既是主串又是模式串，在之前的匹配过程中已经有"t[1]t[2]...t[k-1]"="t[j-k+1]t[j-k+2]...t[j-1]"，则当t[k]≠t[j]时应将模式串向右滑动，比较t[next[k]]和t[j]，若t[next[k]]与t[j]相等，则next[j+1]=next[k]+1,若不相等，继续滑动比较t[next[next[k]]]与t[j]，如不存在任何相等，则next[j+1]=1。
下面举个例子简单的描述一下这个过程（字有点难看）
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/0ee0ed6686626cf5a9d9148c577f9b1c_1740931367206.jpeg)

## next数组求解代码

```cpp
void get_next(string T, int next[])
{
	int i = 1;
	next[1] = 0;
	int j = 0;
	while (i < T.length())
	{
		if (j == 0 || T[i] == T[j])
		{
			++i;
			++j;
			next[i] = j;
		}
		else
			j = next[j];
	}
}
```

## next数组的修正——nextval数组
前面定义的next函数在某些情况下尚有缺陷。例如模式"aaaab"在和主串"aaabaaaab"匹配的过程中，当i=4，j=4时，s[4]≠t[4]，根据next数组还需进行j=3,j=2,j=1这三次比较，实际上这三个字符与第四个字符都是相同的，因此不再需要比较，可以直接比较i=5,j=1。也就是说按上述定义得到next[j]=k，而模式中t[j]=t[k]，则当主串中s[i]≠t[j]时，不需要再和t[k]比较，直接和t[next[k]]比较。

|j|1|2 |3 |4 |5  |
|--|--|--|--|--|--|
|  模式串| a| a| a| a| b|
|next[j]  |0 |1| 2| 3| 4|
|nextval[j]|0 |0 |0 |0 |4|


```cpp
void get_nextval(string T, int nextval[])
{
	int i = 1;
	nextval[1] = 0;
	int j = 0;
	while (i < T.length())
	{
		if (j == 0 || T[i] == T[j])
		{
			i++;
			j++;
			if (T[i] != T[j])
				nextval[i] = j;
			else
				nextval[i] = nextval[j];
		}
		else
			j = nextval[j];
	}
}
```



