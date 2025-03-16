---
title: LeetCode519. 随机翻转矩阵
toc: true
date: 2021-11-27 17:58:16
tags: leetcode 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## LeetCode519. 随机翻转矩阵
[题目传送门](https://leetcode-cn.com/problems/random-flip-matrix/)



## 题目
> 给你一个 m x n 的二元矩阵 matrix ，且所有值被初始化为 0 。请你设计一个算法，随机选取一个满足 matrix[i][j] == 0 的下标 (i, j) ，并将它的值变为 1 。所有满足 matrix[i][j] == 0 的下标 (i, j) 被选取的概率应当均等。
尽量最少调用内置的随机函数，并且优化时间和空间复杂度。
实现 Solution 类：
Solution(int m, int n) 使用二元矩阵的大小 m 和 n 初始化该对象
int[] flip() 返回一个满足 matrix[i][j] == 0 的随机下标 [i, j] ，并将其对应格子中的值变为 1
void reset() 将矩阵中所有的值重置为 0
**数据范围：
1 <= m, n <= 10^4^
每次调用flip 时，矩阵中至少存在一个值为 0 的格子。
最多调用 1000 次 flip 和 reset 方法。**

## 解析
这里m,n的取值上限为10^4^，显然不可能存储所有的数据，但是flip方法最多只会调用1000次，这里容易想到用map来存储已经选取的下标。
首先我们可以把二维数组映射为一个一维数组即下标为（i,j）的元素我们可以用（i*n+j）来表示。

我们模拟的过程是将m×n个0元素排成一行，从前m×n个元素中随机选一个下标将其的值设为1，然后把这个下标和最后一个元素交换，下次选取的时候再从前m×n-1个元素中选取，把最后一个元素（即刚刚选取的元素）去掉，这样保证我们每次选取的元素都是未被选取的。

具体实现方法为：我们将选取的下标的值映射到最后一个元素，这样下次再取到这个下标时，如果这个下标在map中，我们就取它的映射来替换它的值。

**模拟图**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/608f43cd2a876d31378bb13f7153b260_1740931345579.png)

## 代码

```cpp
class Solution
{
public:
    int m, n, total;
    map<int, int> mp;
    Solution(int m, int n)
    {
        this->m = m;
        this->n = n;
        this->total = m * n;
        srand(time(NULL));
    }

    vector<int> flip()
    {
        int idx = rand() % total;
        total--;
        vector<int> ret;
        if (mp.count(idx))
        {
            ret.push_back(mp[idx] / n);
            ret.push_back(mp[idx] % n);
        }
        else
        {
            ret.push_back(idx / n);
            ret.push_back(idx % n);
        }
        if (mp.count(total))
        {
            mp[idx] = mp[total];
        }
        else
        {
            mp[idx] = total;
        }
        return ret;
    }

    void reset()
    {
        total = m * n;
        mp.clear();
    }
};
```

