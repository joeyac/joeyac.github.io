---
title: Gym 101201G Maximum Islands 最大点权独立集
date: 2017-09-14 21:30:18
categories:
  - ACM
tags:
  - 图论
  - 贪心
  - 最大点权独立集
---

给定一个$n × m$的矩阵，每个点可能是陆地、水面或者未定，求最多的陆地联通块的数量。

<!--more-->

## 链接

[G - Maximum Islands](http://codeforces.com/gym/101201)

## 题解

首先对于已经给定的图，有一个明显的贪心是，对于每个陆地，周围的未定点我们都可以看做水面，因为如果将其看做陆地，并不会使得答案更优而且会占用位置，剩下的需要处理的只有若干个未定联通块，对于每个联通块，可以二分图染色，然后未定点的权值设为1，为了方便可以将其他点的权值都设为0，那么就是一个最大点权独立集的问题，套个板子就可以了。

## Something
#### 最小点权覆盖和最大点权独立集

二分图最小点覆盖和最大独立集都可以转化为最大匹配求解。在这个基础上，把每个点赋予一个非负的权值，这两个问题就转化为：二分图最小点权覆盖和二分图最大点权独立集。

#### 二分图最小点权覆盖

从x或者y集合中选取一些点，使这些点覆盖所有的边，并且选出来的点的权值尽可能小。

建模：  原二分图中的边(u,v)替换为容量为INF的有向边(u,v)，设立源点s和汇点t，将s和x集合中的点相连，容量为该点的权值；将y中的点同t相连，容量为该点的权值。在新图上求最大流，最大流量即为最小点权覆盖的权值和。

#### 二分图最大点权独立集

 在二分图中找到权值和最大的点集，使得它们之间两两没有边。其实它是最小点权覆盖的对偶问题。答案=总权值-最小点覆盖集。具体证明参考胡波涛的论文。
 
 ## Update
 本题第二部分似乎用不着二分图最大点权独立集……只需要二分图染色一下，取其中点数多的那半边图即可……
 
 ## 代码
 
 ```cpp
 /*
* Filename:    G.cpp
* Created:     Wednesday, September 13, 2017 09:03:19 AM
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
const int maxn = 40 + 7;

int n, m;
char s[maxn][maxn];
bool vis[maxn][maxn];

int dx[4] = {1, 0, 0, -1};
int dy[4] = {0, -1, 1, 0};

void dfs(int i, int j) {
	vis[i][j] = true;
	for (int k = 0; k < 4; k += 1) {
		int tx = i + dx[k], ty = j + dy[k];
		if (tx < 0 || ty < 0 || tx >= n || ty >= m) continue;
		if (!vis[tx][ty] && s[tx][ty] == 'L') dfs(tx, ty);
	}
}


//  Dinic  //

const int MAX_V=maxn*maxn*2;

struct edge{int to,cap,rev;};
vector<edge>G[MAX_V];
int level[MAX_V];
int iter[MAX_V];
void add_edge(int from,int to,int cap)
{
    edge h;
    h.to=to;
    h.cap=cap;
    h.rev=G[to].size();
    G[from].push_back(h);
    h.to=from;
    h.cap=0;
    h.rev=G[from].size()-1;
    G[to].push_back(h);

}
void bfs(int s)
{
    memset(level,-1,sizeof(level));
    queue<int> que;
    level[s]=0;
    que.push(s);
    while(!que.empty())
    {
        int v=que.front();
        que.pop();
        for(int i=0;i<SZ(G[v]);i++)
        {
            edge &e=G[v][i];
            if(e.cap>0&&level[e.to]<0)
            {
                level[e.to]=level[v]+1;
                que.push(e.to);
            }
        }
    }
}
int dfs(int v,int t,int f)
{
    if(v==t)
        return f;
    for(int &i=iter[v];i<SZ(G[v]);i++)
    {
        edge &e=G[v][i];
        if(e.cap>0&&level[v]<level[e.to])
        {
            int d=dfs(e.to,t,min(f,e.cap));
            if(d>0)
            {
                e.cap-=d;
                G[e.to][e.rev].cap+=d;
                return d;
            }
        }
    }
    return 0;
}
int max_flow(int s,int t)
{
    int flow=0;
    while(1)
    {
        bfs(s);
        if(level[t]<0)
            return flow;
        memset(iter,0,sizeof(iter));
        int f;
        while((f=dfs(s,t,INF))>0)
        {
            flow+=f;
        }
    }
}
// Dinic  //

void build(int start, int end) {
	for (int i = 0; i < n; i += 1)
		for (int j = 0; j < m; j += 1) {
			int v = (s[i][j] == 'C');
			int t = i * m + j + 1;
			if ((i + j) % 2) {
				add_edge(start, t, v);
				if (i > 0) add_edge(t, t - m, INF);
				if (i < n - 1) add_edge(t, t + m, INF);
				if (j > 0) add_edge(t, t - 1, INF);
				if (j < m - 1) add_edge(t, t + 1, INF);
			} else {
				add_edge(t, end, v);
			}
		}
}
int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	//freopen("data.out", "w", stdout);
#endif
	scanf("%d%d", &n, &m);
	for (int i = 0; i < n; i += 1) scanf("%s", s[i]);
	for (int i = 0; i < n; i += 1)
		for (int j = 0; j < m; j += 1)
			if (s[i][j] == 'L') {
				for (int k = 0; k < 4; k += 1) {
					int tx = i + dx[k], ty = j + dy[k];
					if (tx < 0 || ty < 0 || tx >= n || ty >= m) continue;
					if (s[tx][ty] == 'C') s[tx][ty] = 'W';
				}
			}
	clr(vis, 0);
	int ans = 0;
	for (int i = 0; i < n; i += 1)
		for (int j = 0; j < m; j += 1)
			if (s[i][j] == 'L' && !vis[i][j]) dfs(i, j), ans++;
	
	int st = 0, t = n * m + 1;
	build(st, t);
	int res = max_flow(st, t);
	int tol = 0;
	for (int i = 0; i < n; i += 1)
		for (int j = 0; j < m; j += 1)
			if (s[i][j] == 'C') tol++;
	
//	for (int i = 0; i < n; i += 1)
//		printf("%s\n", s[i]);
//	printf("%d %d %d\n", ans, tol, res);

	printf("%d\n", ans + tol - res);
	return 0;
}
 ```
