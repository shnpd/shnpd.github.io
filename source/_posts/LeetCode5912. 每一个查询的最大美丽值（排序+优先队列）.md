---
title: LeetCode5912. 每一个查询的最大美丽值（排序+优先队列）
toc: true
date: 2021-11-14 20:38:19
tags: leetcode 动态规划 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## LeetCode5912. 每一个查询的最大美丽值（排序+优先队列）
[题目传送门](https://leetcode-cn.com/problems/most-beautiful-item-for-each-query/)

## 题目

> 给你一个二维整数数组 items ，其中 items[i] = [pricei, beautyi] 分别表示每一个物品的 价格 和 美丽值 。
同时给你一个下标从 0 开始的整数数组 queries 。对于每个查询 queries[j] ，你想求出价格小于等于 queries[j] 的物品中，最大的美丽值 是多少。如果不存在符合条件的物品，那么查询的结果为 0 。
请你返回一个长度与 queries 相同的数组 answer，其中 answer[j]是第 j 个查询的答案。
提示：
1 <= items.length, queries.length <= 1e5
items[i].length == 2
1 <= pricei, beautyi, queries[j] <= 1e9

给定一组数字，对每一个数字求所有价格小于当前数字的物品中对应的最大的美丽值是多少。
## 分析
这里问对给定的每一个数字，显然如果我们对每一个数字都遍历一遍items数组求和的话必然会超时，要想其他的方法。
对于给定的数字，我们知道大数所能购买的美丽值一定是大于小数的，在这里我们用的方法是首先对queries数组进行排序，每次把当前数字所能购买的所有物品放入一个优先队列中，如果不能购买当前物品，则用下一个数字尝试，这样的话我们不必每次都从头遍历items数组，只需遍历一遍即可。
当我们要求当前数字所对应的最大美丽值，只需输出当前优先队列的头部元素即可。
这里需要注意的是，我们到最后的返回值是要与queries数组一一对应的，但是当我们对queries数组排序时会打乱原数组，这里我们需要对数组的项和索引之间建立一一对应的关系（注意这里不能用值和索引建立，因为值可能会重复），我用的方法是通过pair来保存当前项的值和索引，这样在排序的时候，索引也会一起排列，我们可以通过排序后项的索引来找到它之前的位置。

## 代码

```cpp
class Solution
{
public:
    vector<int> maximumBeauty(vector<vector<int>> &items, vector<int> &queries)
    {
        sort(items.begin(), items.end());
        vector<pair<int, int>> queries2;
        for (int i = 0; i < queries.size(); i++)
        {
            queries2.push_back({queries[i],i});
        }
        sort(queries2.begin(), queries2.end());
        priority_queue<int> pq;
        vector<int> ret(queries.size(), 0);
        int j = 0;
        for (int i = 0; i < queries2.size(); i++)
        {
            int pri = queries2[i].first;
            while (j < items.size()&&items[j][0]<=pri)
            {
                    pq.push(items[j][1]);
                j++;
             }
             if(!pq.empty())
             ret[queries2[i].second]=pq.top();
         }
        return ret;
    }
};
```

