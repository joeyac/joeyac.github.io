---
title: CCPC 2016-2017 Finals - HDU 6005Pandaland
date: 2017-09-16 14:00:53
categories:
  - ACM
tags:
  - 最小生成树
  - LCA
  - 图论
---

给定了 $m(m \leq 4000)$ 条边，每条边有不同的权值$w_{i}$，问其中权值最小的环的权值是多少，如果不存在环，则输出0

<!--more-->

## 链接
[Pandaland](http://acm.hdu.edu.cn/showproblem.php?pid=6005)

## 题解
可以这样想，如果所有的边联通，我们可以按照求最小生成树的方式处理一遍，然后找出未使用的不是重边的最小权值边（这一步再扫一遍排序之后的边即可），加上这条边之后一定可以和原来的最小生成树构成一个最小环，同样的，如果有不连通的边，按照最小生成树的方式处理，最后可以得到多棵最小生成树，那么剩下的没有选取的点肯定也不可能连接任意两棵树，只能是在同一棵树内，于是又规约到单棵树时的问题，求树上两点间的距离可以利用LCA预处理后$O(1)$查询，排序$O(mlogm)$, LCA预处理$O(nlogn)$,本题中$n \leq 2m $, 总复杂度即$O(mlogm)$

## Update
模拟赛赛后去网上找该题的题解，发现题解都是什么dij+剪枝……玄学复杂度卡时间过……并且本题也给了$3s$时限，事实上好像按照我的做法……这道题的边数可以上到$1e5$甚至$1e6$……不明白赛时为啥过的人这么少……赛后杭电上一看，结果我的代码是跑的最快的（第一名第二名都是我233333）
![](http://ow2gecrwu.bkt.clouddn.com/Screenshot%20from%202017-09-16%2014-13-12.png)

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#define clr(a,x) memset(a,x,sizeof(a))
#define pb push_back
#define fi first
#define se second
#define SZ(x) ((int)(x).size())
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
const int maxn = 1e4 + 7;
const int maxm = 4e3 + 7;
const int INF = 1e9 + 7;
typedef pair<int, int> P;

int n, m, T;
struct edge{
    int u,v,w;
    bool fg;
    void set(int _u, int _v, int _w) {
        u = _u; v = _v; w = _w;
        if (u > v) swap(u, v);
        fg = false;
    }
}E[maxm];
vector<P> g[maxn];
map<P, int> M;
map<P, bool> vis;

int F[maxn];
int find(int u) {
    if (F[u] == u) return u;
    return F[u] = find(F[u]);
}
void unite(int u, int v) {
    u = find(u); v = find(v);
    if (u != v) F[v] = u;
}

int dep[maxn], dist[maxn], fa[maxn][20];
void DFS(int u, int _dist, int _dep, int _fa) {
    dist[u] = _dist; dep[u] = _dep; fa[u][0] = _fa;
    for(int i = 0; i < SZ(g[u]); i++) {
        int to = g[u][i].fi, w = g[u][i].se;
        if(to == u || to == _fa) continue;
        DFS(to, _dist + w, _dep + 1, u);
    }
}
void presolve() {
    clr(fa, -1);
    for (int i = 1; i <= n; i++) {
        if(fa[i][0] == -1) DFS(i, 0, 0, i);
    }

    for(int i = 1; i < 20; i++) {
        for(int j = 1; j <= n; j++) {
            fa[j][i] = fa[fa[j][i - 1]][i - 1];
        }
    }
}
int LCA(int u, int v) {
    while(dep[u] != dep[v]) {
        if(dep[u] < dep[v]) swap(u, v);
        int d = dep[u] - dep[v];
        for(int i = 0; i < 20; i++) {
            if(d >> i & 1) u = fa[u][i];
        }
    }
    if(u == v) return u;
    for(int i = 20 - 1; i >= 0; i--) {
        if(fa[u][i] != fa[v][i]) {
            u = fa[u][i];
            v = fa[v][i];
        }
    }
    return fa[u][0];
}
int query(int u, int v) {
    return dist[u] + dist[v] - dist[LCA(u, v)] * 2;
}


void gao() {
    for(int i = 1; i <= n; i++) g[i].clear(), F[i] = i;
    sort(E, E + m, [](edge e1, edge e2){return e1.w < e2.w;});
    vis.clear();
    int st = -1, et = -1;
    for(int i = 0; i < m; i++) {
        int u = E[i].u, v = E[i].v, w = E[i].w;
        if (find(u) == find(v)) continue;
        unite(u, v);
        E[i].fg = true;
        vis[P(u, v)] = true;
        g[u].pb(P(v, w));
        g[v].pb(P(u, w));
    }


    presolve();

    int ans = INF;
    for(int i = 0; i < m; i++) {
        int u = E[i].u, v = E[i].v, w = E[i].w;
        if (vis.count(P(u, v))) continue;
        if (E[i].fg) continue;
        if (find(u) == find(v)) ans = min(ans, w + query(u, v));
    }
    printf("%d\n", ans == INF ? 0 : ans);
}
int main() {
#ifdef AC
    freopen("data.in","r",stdin);
#endif
    int tc = 1, x1, y1, x2, y2, w;
    scanf("%d", &T);
    while (T--) {
        printf("Case #%d: ", tc++);
        n = 0; M.clear();
        scanf("%d", &m);
        for(int i = 0; i < m; i++) {
            scanf("%d%d%d%d%d", &x1, &y1, &x2, &y2, &w);
            if (!M.count(P(x1, y1))) M[P(x1, y1)] = ++n;
            if (!M.count(P(x2, y2))) M[P(x2, y2)] = ++n;
            E[i].set(M[P(x1, y1)], M[P(x2, y2)], w);
        }
        gao();
    }
    return 0;
}
```
