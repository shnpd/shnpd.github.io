---
title: Codeforces Reverse Sort
toc: true
date: 2021-11-14 19:41:18
tags: c++
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## B. Reverse Sort
第一次打codeforces，就做了一道签到题，这道题在看的时候以为挺简单的，看到好多人都出来了，自己想了好久也没做出来，还是看了大佬的解法才有的思路，~~我还是太菜了。~~ 

## 题目：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/b7ba234778735b5c8665b3aa6ad16e8b_1740931345579.png)
大概意思就是给一个01字符串，每次操作可以从中选一个子串做一个翻转然后放回原来的位置，问最小经过几次操作可以让字符串变为非递减的字符串（0都在1的前面）

## 分析
**总可以通过一次操作解决**
~~（太妙了，当时没想到，我是废物）~~ 
我们可以对字符串的0,1进行一个统计，总可以找到一个位置，使得这个位置之前的1的数目等于这个位置之后的0的数目。
例如：101**丨**00，我们可以从竖线处分开，竖线前的1等于竖线后的0，我们只需要把前面的1和后面的0翻转位置，即可得到非递减的字符串。
**为什么我们一定能找到这样一个位置呢？**
我们假设一个位置，在此位置前面的1的数目大于后面的0的数目，那么我们只要把当前位置往前移，如果前面一个是1，那么前面1的数目会减少，后面0的数目不变，如果前面一个是0，那么后面0的数目会增加，前面1的数目不变，由此每向前移动一步，两者之间的差距会减一，当差距减到0的时候，即为要求的位置。

## 代码
先上我自己写的超复杂版
用两个vector分别记录
1.从前往后在当前点之前1的数目
2.从后往前在当前点之后0的数目
然后用flag标记有没有找到上述的位置
如果没有则已经有序，直接输出0即可
```cpp
#include <iostream>
#include <stack>
#include <math.h>
#include <vector>
using namespace std;
int main()
{
    string s;
    int n;
    int T;
    cin >> T;
    while (T--)
    {
        cin >> n;
        cin >> s;
        vector<int> cnt1(n, 0);
        vector<int> cnt0(n, 0);
        int len = n;
        int i;
        bool flag = false;
        for (i = 0; i < len; i++)
        {
            if (s[i] == '1')
            {
                if (i == 0)
                    cnt1[i] = 1;
                else
                    cnt1[i] = cnt1[i - 1] + 1;
            }
            else if (i != 0)
                cnt1[i] = cnt1[i - 1];
        }
        for (i = len - 1; i >= 0; i--)
        {
            if (s[i] == '0')
            {
                if (i == len - 1)
                    cnt0[i] = 1;
                else
                    cnt0[i] = cnt0[i + 1] + 1;
            }
            else if (i != len - 1)
                cnt0[i] = cnt0[i + 1];
        }
        for (i = 1; i < len; i++)
        {
            if (cnt1[i - 1] != 0 && cnt1[i - 1] == cnt0[i])
            {
                cout << 1 << endl;
                flag = true;
                cout << cnt1[i - 1] * 2 << " ";
                for (int j = 0; j <= i - 1; j++)
                {
                    if (s[j] == '1')
                        cout << j + 1 << " ";
                }
                for (int j = i; j < len; j++)
                {
                    if (s[j] == '0')
                        cout << j + 1 << " ";
                }
                cout << endl;
                break;
            }
        }
        if (!flag)
            cout << 0 << endl;
    }
    return 0;
}
```
再上一个大佬的超简单版
[代码来源](https://zhuanlan.zhihu.com/p/432734164)
```cpp
void solve()
{
	int n;cin>>n;
	char c[n+1]; cin>>c;
	vector<int> v;
	for(int i=0;i<n;i++)
	{
	    if(count(c, c+i, '1') == 0) continue;
	    if(count(c, c+i, '1') == count(c+i, c+n, '0'))
	    {
	        cout<<1<<'\n'<<2*count(c, c+i, '1')<<" ";
	        for(int j=0;j<i;j++) if(c[j] == '1') cout<<j+1<<" ";
	        for(int j=i;j<n;j++) if(c[j] == '0') cout<<j+1<<" ";
	        cout<<'\n';
	        return;
	    }
	} 
	cout<<0<<'\n';
}
```


