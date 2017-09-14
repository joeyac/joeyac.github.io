---
title: Gym 101201F Illumination - 2SAT
date: 2017-09-14 21:05:24
categories:
  - ACM
tags:
  - 2-SAT
---

1000*1000的矩形区域内给定$n(n \leq 1000)$盏灯，每盏灯可以横着亮或者竖着亮并照亮该方向上加上自己的$2 l + 1$个格点，要求点亮所有的灯，使得没有两个同一方向上点亮的灯有重合照亮的位置。

<!--more-->

## 链接

[F - Illumination](http://codeforces.com/gym/101201)

## 题解

可以发现每盏灯只有两个状态横着点亮或者竖着点亮，用$0/1$表示，那么可以$O(n^2)$枚举两盏灯在某个点亮方向上是否和另外一盏灯冲突，这个限制即两盏灯的状态不能同时为$0或1$，由这个逻辑关系，剩下的就是2-SAT的事了。

## 代码

```cpp
/*
* Filename:    2-SAT.cpp
* Created:     Wednesday, September 13, 2017 04:06:32 PM
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
const int maxn = 1e3 + 7;

struct Twosat {
	int n;
	vector<int> g[maxn * 2];
	bool mark[maxn * 2];
	int S[maxn * 2], c;
	
	bool dfs(int x) {
		if (mark[x ^ 1]) return 0;
		if (mark[x]) return 1;
		mark[x] = 1;
		S[c++] = x;
		for (int i = 0; i < SZ(g[x]); i += 1)
			if (!dfs(g[x][i])) return 0;
		return 1;
	}
	
	void init(int n) {
		this->n = n;
		for (int i = 0; i < n * 2; i += 1) g[i].clear();
		clr(mark, 0);
	}
	
	// x=xval || y=yval
	void add_clause(int x, int xval, int y, int yval) {
		x = x * 2 + xval;
		y = y * 2 + yval;
		g[x ^ 1].pb(y);
		g[y ^ 1].pb(x);
	}
	
	bool solve() {
		for (int i = 0; i < n * 2; i += 2) {
			if (!mark[i] && !mark[i + 1]) {
				c = 0;
				if (!dfs(i)) {
					while (c > 0) mark[S[--c]] = 0;
					if (!dfs(i + 1)) return 0;
				}
			}
		}
		return 1;
	}
}tsat;

int n, m, R;
int r[maxn], c[maxn];

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	//freopen("data.out", "w", stdout);
#endif
	scanf("%d%d%d", &n, &R, &m);
	tsat.init(m);
	for (int i = 0; i < m; i += 1) scanf("%d%d", &r[i], &c[i]);
	for (int i = 0; i < m; i += 1)
		for (int j = i + 1; j < m; j += 1) {
			if (r[i] == r[j] && abs(c[i] - c[j]) <= R)
				tsat.add_clause(i + 1, 0, j + 1, 0);
			if (c[i] == c[j] && abs(r[i] - r[j]) <= R)
				tsat.add_clause(i + 1, 1, j + 1, 1);
		}
	if (tsat.solve()) puts("YES");
	else puts("NO");
	return 0;
}
```
