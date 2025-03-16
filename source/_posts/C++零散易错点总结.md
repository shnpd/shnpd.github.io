---
title: C++零散易错点总结
toc: true
date: 2021-08-10 09:50:03
tags: c++
categories: 杂项
---

​​点击阅读更多查看文章内容<!--more-->

## 对日常做题中遇到的一些零散的易错点的总结
**持续更新ing**
1.string的length方法返回的是无符号数，当与负数比较时需要强制类型转换，否则会报错。**-1<s.length()   ->   -1<(int)s.length()**
2.关于运算符重载，在使用优先队列时，特别注意第二个const不能省略，否则会报错。**bool operator<(const type name)const**
3.遍历map的方法：**for (auto& [ key , value ] : map)**
4.二维向量排序：**sort(people.begin(), people.end(), [](const vector<int> &u, const vector<int> &v)
             { return u[0] < v[0] || (u[0] == v[0] && u[1] > v[1]); });**
5.
c++如下声明向量会报错，原因是编译器可能无法区分这是一个成员函数声明还是一个成员变量声明。
```
class Solution
{
public:
    vector<int> F (40, 0);
    double myPow(double x, int n)
    {
    	return -1;
    }
};
```
解决方法是
```
vector<int> F = vector<int>(40,0);//明确这是一个成员变量声明
```
