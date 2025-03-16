---
title: LeetCode629.K个逆序对数组（dp）
toc: true
date: 2021-11-11 21:32:54
tags: leetcode 动态规划 算法
categories: 刷题集
---

​​点击阅读更多查看文章内容<!--more-->

## LeetCode629.K个逆序数组（dp）
[题目传送门](https://leetcode-cn.com/problems/k-inverse-pairs-array/)

> 给出两个整数 n 和 k，找出所有包含从 1 到 n 的数字，且恰好拥有 k 个逆序对的不同的数组的个数。
逆序对的定义如下：对于数组的第i个和第 j个元素，如果满i < j且 a[i] > a[j]，则其为一个逆序对；否则不是。
由于答案可能很大，只需要返回 答案 mod 109 + 7 的值。

## 解析
**本题主要参考的官方题解，但是官方题解有些地方当时理解的时候有些困难，在这里再记录一下，便于理解。**
[官方题解](https://leetcode-cn.com/problems/k-inverse-pairs-array/solution/kge-ni-xu-dui-shu-zu-by-leetcode-solutio-0hkr/)
f[i][j]表示长度为i的数组，恰好包含j个逆序对的方案数，第i个元素的所有可能取值为1~i中的一个数字，我们假设第i个元素为k，那么数组中逆序对的个数为一下两部分之和：
1.数字k与另外i-1个元素产生的逆序对的个数
2.另外i-1个元素内部产生的逆序对的个数

对第一部分：数字k放在最后一位，数组中有i-k个比数字k大的数，所以k贡献的逆序对个数为i-k
对第二部分：因为f[i][j]表示的有j个逆序对的情况，所以我们希望第二部分的逆序对个数为j-(i-k)，这里逆序对的个数只与元素的相对大小有关，不包含k的数组元素为1,...,k-1和k+1,...i。我们可以把后半部分整体减一，此时逆序对的个数不变，我们的目标变成了1,...i-1，希望它有j-(i-k)个逆序对，由此我们可以得到状态转移方程。
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/ebbe4b4fec409e9ea7dbfcf9750ddfee_1740931345579.png)
边界条件为：
f[0][0]=1，不用任何数字构成一个空数组，包含0个逆序对
f[i][j<0]=0，逆序对的数量一定是非负整数
最终的答案为f[n][k]

## 优化
**这里的优化非常巧妙，需要着重理解**
首先上述动态规划的状态数量为O(nk)，而求出每一个f[i][j]需要经过O(n)的时间复杂度，总时间复杂度会超时
注意f[[i][j-1]和f[i][j]的状态转移方程：
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/f1fd20c24b967e3766cd89d21a3a66ae_1740931345579.png)
**简单展开
f[i][j]=f[i-1][j]+f[i-1][j-1]+...+f[i-1][j-i+1]
f[i][j-1]=f[i-1][j-1]+f[i-1][j-2]+...+f[i-1][j-i]
不难看出
f[i][j]=f[i][j-1]+f[i-1][j]-f[i-1][j-i]
这样我们就可以在常数时间内求出f[i][j]**
此外，我们可以发现，对f[i][j]的求解只会用到f[i][...]和f[i-1][...]，因此我们可以再对空间进行优化，用两个一维数组交替进行状态转移，**此步对应代码中now=i&1,prev=now^1，now表示当前要求的数组，prev则是前一个数组，now随着i增大是1,0交替变化，prev则是随着now的变化0,1交替变化，最开始now为1，此时prev为0，当i增加时，now变为0，prev变为1，此时dp[prev]的值即是上一个now的值，由此实现两数组的交替状态转移。**

## 代码

```cpp
class Solution
{
public:
    int kInversePairs(int n, int k)
    {
        int mod = 1e9 + 7;
        int dp[2][1010];
        memset(dp, 0, sizeof(dp));
        dp[0][0] = 1;
        for (int i = 1; i <= n; i++)
        {
            int now = i & 1, prev = now ^ 1;
            for (int j = 0; j <= k; j++)
            {
                dp[now][j] = (j - 1 >= 0 ? dp[now][j - 1] : 0) - (j - i >= 0 ? dp[prev][j - i] : 0) + dp[prev][j];
                if (dp[now][j] >= mod)
                    dp[now][j] -= mod;
                else if (dp[now][j] < 0)
                    dp[now][j] += mod;
            }
        }
        return dp[n & 1][k];
    }
};
```

