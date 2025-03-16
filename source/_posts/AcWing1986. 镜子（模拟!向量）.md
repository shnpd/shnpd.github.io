---
title: AcWing1986. 镜子（模拟/向量）
toc: true
date: 2022-04-29 23:26:12
tags: 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

[题目链接](https://www.acwing.com/problem/content/description/1988/)

# 题目分析
小模拟题，第一遍做的时候越做越复杂，看了y总的题解，发现y总的这种做题方法还蛮不错的，条理清晰，先在main函数中把代码的总体框架敲好，比较复杂或者使用比较多的方法写成函数，这里的函数可以先不去声明，直接在main函数中写，等main函数的总体框架写完之后再去完善其他的函数

**总体思路**
本题的N最大只有200，判断时间复杂度可以到达O(N^3^)
我们首先枚举要变换的镜子
然后镜子最多只会反射2*(n+1)次
然后再遍历所有镜子找到在当前镜子的方向上距离最近的那个
总的时间复杂度为O(N^3^)

这里着重解释一下找在当前镜子的方向上距离最近的那个镜子的代码

```cpp
  //判断j是否在k定义的方向上    
  if (m[k].x + abs(m[j].x - m[k].x) * fx[d] != m[j].x)
  		continue;
  if (m[k].y + abs(m[j].y - m[k].y) * fy[d] != m[j].y)
        continue;
```
原理如下：              
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ced3d531735ce20bc3b68b1eee128ca6_1740930605920.png%20=300x)
计算距离的代码，这个代码只有当线段平行于坐标轴时才能使用
```cpp
 int dis = abs(m[k].x - m[j].x) + abs(m[k].y - m[j].y);
```

# 代码
```cpp
#include <iostream>
#include <cstring>
#include <string>
#include <algorithm>

using namespace std;

const int N = 210, INF = 1e8;

struct Mirror
{
    int x, y;
    char c;
} m[N];
int n;
int fx[4] = {0, 1, 0, -1};
int fy[4] = {1, 0, -1, 0};
bool check()
{
    int d = 1;//方向
    int k = 0;//当前的镜子
    //最多会反射2*(n+1)次
    for (int i = 0; i < 2 * (n + 1); i++)
    {
        int id = -1;//在当前方向上最近的镜子
        int len = INF;
        //找到当前方向上最近的镜子
        for (int j = 1; j <= n + 1; j++)
        {
            if(k==j)
                continue;
            //判断j是否在k定义的方向上    
            if (m[k].x + abs(m[j].x - m[k].x) * fx[d] != m[j].x)
                continue;
            if (m[k].y + abs(m[j].y - m[k].y) * fy[d] != m[j].y)
                continue;

            int dis = abs(m[k].x - m[j].x) + abs(m[k].y - m[j].y);
            if (dis < len)
            {
                len = dis;
                id = j;
            }
        }
        if (id == n + 1)
            return true;
        if (id == -1)
            return false;
        k = id;
        if (m[k].c == '\\')
            d ^= 3;
        else
            d ^= 1;
    }
    return false;
}
void rotate(char &c)
{
	//'\'需要转义
    if (c == '/')
        c = '\\';
    else
        c = '/';
}
int main()
{
    cin >> n;
    cin >> m[n + 1].x >> m[n + 1].y;
    for (int i = 1; i <= n; i++)
        cin >> m[i].x >> m[i].y >> m[i].c;
    if (check())
    {
        cout << 0;
        return 0;
    }
    for (int i = 1; i <= n; i++)
    {
        rotate(m[i].c);
        if (check())
        {
            cout << i<<endl;
            return 0;
        }
        rotate(m[i].c);
    }
    cout << -1;
    return 0;
}

```

