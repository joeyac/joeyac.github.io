---
title: 'cf #474 F.Pathwalks - 整体二分'
categories:
  - ACM
tags:
  - 整体二分
date: 2018-04-08 23:15:45
---

给定$10^5$条边，需要找一条路径，使得路径上的边的权值严格递增，并且边的编号也是严格递增的，求最长路径包含的边的数量。

<!--more-->

## 链接
[Pathwalks](http://codeforces.com/problemset/problem/960/F)

## 题解
似乎正解是DP，不过赛时没想这么多，因为这题有很显然的两维偏序关系，一个即输入的边的id，一个是输入的边的权值，于是想着能不能用整体二分试试，显然如果知道以某个点为入点的最大后继路径的长度，那么就可以用这个来更新所有以这个点为出点的边的后继路径长度，同时需要注意一下边权的限制，这个就可以直接用整体二分来做了。

再考虑一些细节，由于每条边只能和编号比自己大的点构成路径，那么在整体二分中，应该是先处理右半边区间，再处理自己，再处理左半边区间，在这个过程中需要复原操作（即复原输入的边集），利用类似归并排序的思路即可，利用inplace_merge或者merge函数可以很方便的完成这个工作。

## 代码

```cpp
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
#define cmin(a, b) ((a) = ((a) < (b) ? (a) : (b)))
#define cmax(a, b) ((a) = ((a) > (b) ? (a) : (b)))

using namespace std;

typedef long long ll;
typedef pair<int, int> P;

const int inf = 1e9 + 7;
const int mod = 1e9 + 7;
const int maxn = 1e5 + 7;

int n, m, k;
struct edge {
	int id, a, b, w, ans;
	void in(int i) {
		id = i;
		scanf("%d%d%d", &a, &b, &w);
		this->ans = 1;
	}
	void out() {
		printf("%d: a=%d,b=%d,w=%d,ans=%d\n", id, a, b, w, ans);
	}
}e[maxn], e1[maxn], e2[maxn];
bool cmp(const edge &e1, const edge &e2) {
	return e1.id < e2.id;
}

void divide(int ql, int qr, int mid, int &c1, int &c2) {
	for (int i = ql; i <= qr; i++) {
		if (e[i].w <= mid) e1[c1++] = e[i];
		else e2[c2++] = e[i];
	}
	for (int i = 0; i < c1; i++) e[ql + i] = e1[i];
	for (int i = 0; i < c2; i++) e[ql + c1 + i] = e2[i];
}

int ans[maxn];

void sol(int ql, int qr, int l, int r) {
	if (ql >= qr || l >= r) return;
	int mid = (l + r) >> 1;
	
	int c1 = 0, c2 = 0;
	divide(ql, qr, mid, c1, c2);
	sol(ql + c1, qr, mid + 1, r);
	inplace_merge(e + ql, e + ql + c1, e + qr + 1, cmp);

	for (int i = qr; i >= ql; i--)
		if (e[i].w <= mid) cmax(e[i].ans, ans[e[i].b] + 1);
		else  cmax(ans[e[i].a], e[i].ans);
	for (int i = qr; i >= ql; i--) 
		if (e[i].w > mid) ans[e[i].a] = 0;
	
	c1 = c2 = 0;
	divide(ql, qr, mid, c1, c2);
	sol(ql, ql + c1 - 1, l, mid);
	inplace_merge(e + ql, e + ql + c1, e + qr + 1, cmp);
}

int main() {
#ifdef AC
    freopen("data.in", "r", stdin);
#endif
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= m; i++) e[i].in(i);
	sol(1, m, 0, 1e5);
	int out = 0;
	for (int i = 1; i <= m; i++) cmax(out, e[i].ans);
	printf("%d\n", out);
	return 0;
}
```