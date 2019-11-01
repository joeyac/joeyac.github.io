---
title: aoj0525 Osenbei (bitset的使用)
tags:
  - bitset
  - 暴力
  - 枚举
id: 222
categories:
  - ACM
date: 2016-03-10 17:18:03
---

链接:[aoj0525](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=0525)
> 题意：药药!切克闹! 煎饼果子来一套!有一个烤饼器可以烤r行c列的煎饼，煎饼可以正面朝上（用1表示）也可以背面朝上（用0表示）。一次可将同一行或同一列的煎饼全部翻转。现在需要把尽可能多的煎饼翻成正面朝上，问最多能使多少煎饼正面朝上？
> 
> 输入：多组输入，每组第一行为二整数r, c (1 ≤ r ≤ 10, 1 ≤ c ≤ 10 000)，剩下r行c列表示煎饼初始状态。r=c=0表示输入结束。
> 
> 输出：对于每组输入，输出最多能使多少煎饼正面朝上。
> 
> 
> 中文参考http://bbs.byr.cn/#!article/ACM_ICPC/73337?au=Milrivel
第一想法其实就是枚举....没有其他什么,不过感觉不是很好处理.....直接暴力大概会超时吧也没去试,

看到大牛博客的关于bitset的使用,才知道还有个这么神奇的东西,顺便转个文章做个总结~

[传送门](http://jingwei.site/bitset/)

因为r最大只有10,只需要预先建立10个bitset即可,处理起来很方便.

    /*
    * Filename:    aoj0525.cpp
    * Desciption:  Desciption
    * Created:     2016-03-10
    * Author:      JIngwei Xu [mail:xu_jingwei@outlook.com]
    *
    */
    #include<bitset>
    #include<iostream>
    #include<stdio.h>
    #include<algorithm>
    #include<cstring>
    #define INT_MAX 1<<30
    using namespace std;
    typedef long long ll;
    const int INF=0x7F;
    int n,m,ans;
    bitset<10000> bits[10];
    int check(){
        int ct=0,rec=0;
        for (int i = 0; i < m; i += 1)
        {
            ct=0;
            for (int j = 0; j < n; j += 1)
            {
                if(bits[j][i]==1)ct++;
            }
            ct=max(ct,n-ct);
            rec+=ct;
        }
        return rec;
    }
    void dfs(int k){
        if (k==n)
        {
            ans=max(ans,check());
            return;
        }
        for (int i = 0; i < 2; i += 1)
        {
            bits[k].flip();
            dfs(k+1);
        }
    }
    int main()
    {
    //ios_base::sync_with_stdio(0);
    #ifdef JIngwei_Xu
        freopen("data.in","r",stdin);
        freopen("data.out","w",stdout);
    #endif
        while (scanf("%d%d",&n,&m)&&n&&m)
        {
            int t;
            ans=0;
            for (int i = 0; i < n; i += 1)
            {
                for (int j = 0; j < m; j += 1)
                {
                    scanf("%d",&t);
                    bits[i][j]=t;
                }
            }
            dfs(0);
            printf("%d\n",ans);
        }
        return 0;
    }
