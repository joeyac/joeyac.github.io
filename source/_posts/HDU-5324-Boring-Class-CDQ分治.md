---
title: HDU - 5324 - Boring Class - CDQ分治
categories:
  - ACM
tags:
  - CDQ分治
  - 线段树
date: 2017-10-03 21:41:28
---


简单来说，给出n个二维点对，求$LIS$长度和编号字典序最小的$LIS$（$x$非增，$y$非减），并输出最优$LIS$。

显然有一个$DP[i] = \max\{DP[j] + 1\ | \ i > j, L[i] \geq L[j], R[i] \leq R[j] \}$

那么就是一个三维偏序问题。

<!--more-->

## 链接
[Boring Class](http://acm.hdu.edu.cn/showproblem.php?pid=5324)

## 题解
首先$DP[i] = \max\{DP[j] + 1\ | \ i > j, L[i] \geq L[j], R[i] \leq R[j] \}$， 然后很明显的三维偏序问题，直接上CDQ没跑，sort一下$L[i]$，然后cdq中处理一下下标$i$,剩下的需要做的是，找一个数据结构维护$(R, value, id)$ （value为DP值，id维下标）这样一个三元组，同时对于某个$R_x$,查询所有满足$R \leq R_x $的三元组中$value$最大并且$id$最小(为了字典序最小？)的一个，这个可以考虑用线段树做，线段树直接存一个$pair<int, int>$，为了使得$id$最小，将$id$取负存入然后直接用线段树做单点更新和区间查询即可。

但是现在还有一个问题需要仔细思考，如果用上述方式$DP$,并且记录最优转移路线，那么只能保证每个点的前驱是最小的字典序，这样并不能保证输出的字典序最小，所以$DP$式子应该换一下:

$DP[i] = \max\{DP[j] + 1\ | \ i < j, L[i] \geq L[j], R[i] \leq R[j] \}$

倒过来$DP$,并且同时记录最优转移路线，这样通过记录后继，就可以保证求出来的$LIS$字典序最小。

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef pair<int, int> P;
const int maxn = 5e4 + 7;
const int INF = 1e9 + 7;

namespace Seg{
	#define lson l, m, rt << 1
	#define rson m + 1, r, rt << 1 | 1
	#define args int l = 1, int r = N, int rt = 1
	int N;
	P dat[maxn << 2];
	bool clr[maxn << 2];
	void clean() {clr[1] = true; dat[1] = P(0, 0);}
	void pu(int rt) {dat[rt] = max(dat[rt << 1], dat[rt << 1 | 1]);}
	void pd(int rt) {
		if (clr[rt]) {
			dat[rt << 1] = dat[rt << 1 | 1] = P(0, 0);
			clr[rt << 1] = clr[rt << 1 | 1] = true;
			clr[rt] = false;
		}
	}
	void update(int R, int value, int id, args) {
		if (l == r) dat[rt] = max(dat[rt], P(value, -id));
		else {
			pd(rt);
			int m = (l + r) >> 1;
			if (R <= m) update(R, value, id, lson);
			else update(R, value, id, rson);
			pu(rt);
		}
	}
	P query(int L, int R, args) {
		if (L <= l && r <= R) return dat[rt];
		pd(rt);
		int m = (l + r) >> 1;
		P ret(0, 0);
		if (L <= m) ret = max(ret, query(L, R, lson));
		if (R > m) ret = max(ret, query(L, R, rson));
		pu(rt);
		return ret;
	}
};
int n, m, tmp[maxn];
struct oper{
	int id, l, r, val;
	int pre;
}q[maxn], q1[maxn], q2[maxn];
bool cmp(const oper& o1, const oper& o2) {
	if (o1.l == o2.l) return o1.id < o2.id;
	return o1.l > o2.l;
}
bool cmp2(const oper& o1, const oper& o2) {return o1.id < o2.id;}

void divide(int l, int r) {
	int c1 = 0, c2 = 0, mid = (l + r) >> 1;
	for (int i = l; i <= r; i++) if (q[i].id <= mid) q1[c1++] = q[i];
	else q2[c2++] = q[i];

	for (int i = 0; i < c1; i += 1)q[i + l] = q1[i];
	for (int i = 0; i < c2; i += 1)q[i + l + c1] = q2[i];

}

void cdq(int l, int r) {
	if (l == r) return;
	int mid = (l + r) >> 1;
	divide(l, r);
	cdq(mid + 1, r);
	sort(q + l, q + r + 1, cmp);
	Seg::clean();
	for (int i = r; i >= l; i += -1) if (q[i].id > mid) Seg::update(q[i].r, q[i].val, q[i].id);
	else {
		P x = Seg::query(q[i].r, Seg::N);
		int value = x.first + 1, id = -x.second;
		if (value > q[i].val) q[i].val = value, q[i].pre = id;
		else if (value == q[i].val && id < q[i].pre) q[i].pre = id;
	}
	
	divide(l, r);
	cdq(l, mid);
}

int main() {
//	freopen("data.out", "r", stdin);

	while (scanf("%d\n", &n) != EOF) {
		for (int i = 1; i <= n; i++) scanf("%d", &q[i].l);
		for (int i = 1; i <= n; i++) scanf("%d", &q[i].r);
		for (int i = 1; i <= n; i++) q[i].id = i, q[i].val = 1, q[i].pre = -1;
		
		for (int i = 1; i <= n; i += 1) tmp[i - 1] = q[i].r;
		sort(q + 1, q + n + 1, cmp);
		sort(tmp, tmp + n);
		m = unique(tmp, tmp + n) - tmp;
		for (int i = 1; i <= n; i++) q[i].r = (lower_bound(tmp, tmp + m, q[i].r) - tmp) + 1;
		Seg::N = m;
		
		cdq(1, n);
		sort(q + 1, q + n + 1, cmp2);
		int ans = 0, id = 0;
		
		for (int i = 1; i <= n; i++)
			if (q[i].val > ans) ans = q[i].val, id = i;
		
		printf("%d\n", ans);
		vector<int> vc; vc.clear();
		while (~id) {
			vc.push_back(id);
			id = q[id].pre;
		}
		assert((int)vc.size() == ans);
		m = vc.size();
		for (int i = 0; i < m; i++) printf("%d%c", vc[i], " \n"[i == m - 1]);
	}
	return 0;
}

```