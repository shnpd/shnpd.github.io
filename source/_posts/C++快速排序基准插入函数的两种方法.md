---
title: C++快速排序基准插入函数的两种方法
toc: true
date: 2021-09-14 18:39:21
tags: c++ 排序算法 快速排序
categories: 算法
---

​​点击阅读更多查看文章内容<!--more-->

## C++快速排序基准插入函数的两种方法

今天做题的时候刷到一种新的快排的实现方法，主要是基准插入函数的不同，这种方法相比于常规方法时间复杂度较高，但是比较容易理解，在这里记录一下。

## 第一种（最常见的双指针遍历）
```cpp
int partition(vector<int>&arr, int i, int j) {
    int key = arr[i];
    while (i < j) {
        while (i < j && arr[j] >= key) {
            j--;
        }
        if (i < j) {
            arr[i] = array[j];
        }
        while (i < j && arr[i] <= key) {
            i++;
        }
        if (i < j) {
            arr[j] = array[i];
        }
    }
    arr[i] = key;
    return i;
}
```

## 第二种（从头到尾遍历一遍，把小于基准值的依次与最前面的交换）

```cpp
int partition(vector<int> &arr, int l, int r)
    {
        int key = arr[l];
        int j = l;
        for (int i = l + 1; i <= r; i++)
        {
            if (arr[i] < key)
            {
                j++;
                swap(arr[i], arr[j]);
            }
        }
        swap(arr[j], arr[l]);
        return j;
    }
```

