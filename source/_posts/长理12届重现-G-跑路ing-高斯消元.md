---
title: 长理12届重现-G-跑路ing-高斯消元
date: 2017-09-03 21:56:41
categories:
  - ACM
tags:
  - 高斯消元
---

给定一个n个点m条边的有向图$(1<=n<=100, 1<=m<=10000)$，从1号点出发，每单位时间必须从目前位置等概率选择一条边然后移动到对应的节点上去或者不动（如果当前节点有t条边，则有1/(t+1)的概率选择一条边移动或者原地不动），可以认为每次需要花费1 单位时间。

保证在每个位置的概率收敛，求最大概率的那个位置的概率。

<!-- more -->

## 链接
[G-跑路ing](https://www.nowcoder.com/acm/contest/submit/fd8e9451406b4ef8bdeb27c0e2d4cd6b?ACMContestId=1&tagId=4)

## 题解
首先我们可以把从一号点可以到达的点都拎出来重新编号为$[1, c]$，考虑$DP[i]$代表编号之后$i$号节点最终的概率，那么有：

$$ DP[i] = \sum_{j \in A} \frac{DP[j]}{SZ(j)+1} $$

其中$A$集合代表所有能到达点$i$的点的集合，$SZ(j)$代表从$j$向外连出的边的数量。

共有$c$个这样的式子，将这些式子移项整理，即得出$c$个方程，再加上
$$ \sum_{i=1}^{c} DP[i]= 1 $$
共$c+1$个方程，$c$个未知数，由于数据保证收敛，即该方程组有解，套一个高斯消元的板子即可。需要注意的一个坑点是会有重边和自环。

## 代码
```cpp
/*
* Filename:    G.cpp
* Created:     Sunday, September 03, 2017 03:59:46 PM
* Author:      crazyX
* More:
*
*/
#include <bits/stdc++.h>

#define mp make_pair
#define pb push_back
#define fi first
#define se second
#define SZ(x) ((int) (x).size())
#define all(x) (x).begin(), (x).end()
#define sqr(x) ((x) * (x))
#define clr(a,b) (memset(a,b,sizeof(a)))
#define y0 y3487465
#define y1 y8687969
#define fastio std::ios::sync_with_stdio(false)

using namespace std;

typedef long long ll;

const int INF = 1e9 + 7;
const int maxn = 100 + 7;

////////////Gauss///////////
//接口：Gauss();解存在x数组中。equ，var分别表示方程个数和变量的个数，（因为是变量 的个数不算常数的个数）要赋值。a就是行列式，其中常数是在方程右侧的符号。  
const double eps=1e-10;  
double a[maxn][maxn],x[maxn];  
int equ,var;  
int Gauss()  
{  
    int i, j, k, max_r, col;  
    double tmp;  
    col = 0;  

    for(k = 0; k<equ && col<var; k++, col++)  
    {  
        max_r = k;  
        for(i = k+1; i < equ; i++)  
            if(fabs(a[i][col])-fabs(a[max_r][col]) > eps)  
            max_r = i;  

        if(max_r != k)  
            for(j = k; j < var+1; j++)  
            swap(a[k][j], a[max_r][j]);  

        if(fabs(a[k][col]) < eps)  
        {  
            k--;  
            continue;  
        }  
        for(i = k+1; i < equ; i++)  
        {  
            if(fabs(a[i][col]) > eps)  
            {  
                double t = a[i][col]/a[k][col];  //这里和整型的不同。  
                a[i][col] = 0.0;  

                for(j = col; j < var+1; j++)  
                a[i][j] -= a[k][j]*t;  
            }  
        }  
    }  
    for(i = var-1; i >= 0; i--)  
    {  
        if(fabs(a[i][i]) < eps) continue;  
        tmp = a[i][var];  
        for(j = i+1; j < var; j++)  
        if(a[i][j] != 0)  
        tmp -= a[i][j]*x[j];  

        //if(tmp%a[i][i] != 0) return -2;  
        x[i] = tmp/a[i][i];  
    }  
    return 0;  
}  
////////////Gauss///////////



int n, m, idx[maxn], rid[maxn], tot, u, v;
bool vis[maxn];

vector<int> g[maxn], rg[maxn];

void dfs(int x) {
    if(vis[x]) return;
    vis[x] = true;
    idx[x] = tot;
    rid[tot] = x;
    tot++;
    for (auto y : g[x]) dfs(y);
}

int main()
{
#ifdef AC
    freopen("data.in", "r", stdin);
    //freopen("data.out", "w", stdout);
#endif
    while (scanf("%d%d", &n, &m) != EOF) {
        for (int i = 1; i <= n; i += 1) g[i].clear(), rg[i].clear();
        for (int i = 0; i < m; i += 1) {
            scanf("%d%d", &u, &v);
            g[u].pb(v); rg[v].pb(u);
        }
        clr(vis, 0); tot = 0;
        dfs(1);

        equ = var = tot;

        for (int i = 0; i < equ; i += 1) {
            for (int j = 0; j < var; j += 1) a[i][j] = 0;

            a[i][i] = 1.0 - 1.0 / (SZ( g[rid[i] ]) + 1);
            for (auto y : rg[ rid[i] ]) {
                a[i][idx[y]] += -1.0 / (SZ(g[y]) + 1);
            }
        }
        for (int i = 0; i < equ; i += 1) a[i][var] = 0;

        for (int j = 0; j < var; j += 1) a[equ][j] = 1;
        a[equ][var] = 1;
        equ++;

//		for (int i = 0; i < equ; i += 1) 
//			for (int j = 0; j <= var; j += 1)
//				printf("%.2f%c",a[i][j], " \n"[j==var]);

        Gauss();
        double ans = 0;
        for (int i = 0; i < var; i += 1) ans = max(ans, x[i]);
        printf("%.8f\n", ans);
    }
    return 0;
}
```