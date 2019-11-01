---
title: CodeForces Gym 101190 B - Binary Code
mathjax: true
categories:
  - ACM
tags:
  - 字典树
  - Trie
  - 2-SAT
  - SCC缩点
date: 2017-09-22 12:29:54
---

给定$n (n \leq 5e5, \sum length \leq 5e5)$个01字符串,每个字符串<b>最多</b>包含一个问号，问是否存在一种将所有问号都用$0 /\ 1$替代的方案，并且使得$n$个字符串任意两个都互不是对方的前缀。

<!--more-->

## 链接

[Binary Code](http://codeforces.com/gym/101190)

## 题解

既然只有$0 /\ 1$,那么很自然的会联想到可能使用Trie树搞一搞，然后因为每个问号只会被替换成0或1，又可以想到用2-SAT来处理。

首先可以将字符串插入到Trie中，对于含有问号的字符串，把问号用0、1替换之后分别插入Trie中，对于每个字符串的终点，往对应的Trie树上的节点插入该字符串的id。

首先考虑特殊一点的情况，每个Trie上的节点如果最多只有一个字符串的终点，那么由这个节点往上走，一直到Trie的根节点，是不能出现其他字符串的终点的，这里可以用经典的2-SAT建模的方式维护。

接下来考虑稍微一般的情况，对于一个Trie上的节点，如果有多个字符串的终点，显然最多只有一个点可以为$true$，那么可以维护一下这若干个点的前缀和以及后缀和，具体做法是通过分别引入k个点作为辅助节点来实现的，下图以四个点为例：

![](http://ow2gecrwu.bkt.clouddn.com/170922-135755.jpg)

![](http://ow2gecrwu.bkt.clouddn.com/170922-135746.jpg)

对于四个点的情况如图，其中$H 点和 D 点$即Trie上这个节点的出口$st[j]$和入口$ed[j]$，这样的话，我们就可以遍历每个字符串的终端节点$i$的所有非空父节点$j$，然后$i -> st[j]$建边，$ed[j] -> i \wedge 1$ 建边即可。

然后就可以对原图进行scc缩点，缩点之后分别判断n个字符串的两个变量值所对应的点是否在一个联通块内，如果都不在，那么将缩点之后的新图求一个拓扑序并染色即可。

## 代码

```cpp
/*
* Filename:    B-final.cpp
* Created:     Thursday, September 21, 2017 03:25:19 PM
* Author:      crazyX
* More:        2-SAT trie优化建图 前缀和 后缀和
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
const int maxn = 1e6 + 7;
const int maxm = 3e6 + 7;

char s[maxn], outs[maxn];
int sst[maxn], sed[maxn], sn;
int n, m, N;

vector<int> g[maxm];


int tol;
int nxt[maxn][2], pre[maxn], re[maxn];
vector<int> vc[maxn];

void add(int id) {
	int cur = 0;
	for (int i = 0; i < m; i += 1) {
		int dir = s[i] - '0';
		if (!nxt[cur][dir]) {
			nxt[cur][dir] = ++tol;
			pre[tol] = cur;
		}
		cur = nxt[cur][dir];
	}
	re[id] = cur;
	vc[cur].pb(id);
}

int st[maxn], ed[maxn];
void generate() {
	N = n << 1;
	
	for (int i = 1; i <= tol; i += 1) {
		if (vc[i].empty()) continue;
		for (int j = 0; j < SZ(vc[i]); j += 1)
			if (!j) g[N++].pb(vc[i][j] ^ 1);
			else g[vc[i][j]].pb(N - 1), g[N].pb(N - 1), g[N++].pb(vc[i][j] ^ 1);
		
		st[i] = N - 1;
		for (int j = 0; j < SZ(vc[i]); j += 1)
			if (!j) g[vc[i][j]].pb(N++);
			else g[N - 1].pb(vc[i][j] ^ 1), g[N - 1].pb(N), g[vc[i][j]].pb(N++);
		
		ed[i] = N - 1;
	}
	
	for (int i = 0; i < (n << 1); i += 1)
		for (int j = pre[re[i]]; j ; j = pre[j]) if (!vc[j].empty())
			g[i].pb(st[j]), g[ed[j]].pb(i ^ 1);
}


 
int tp, tim, num, scc[maxm], sta[maxm], low[maxm], dfn[maxm];

void dfs(int x) {
	dfn[x] = low[x] = ++tim;
	sta[++tp] = x;
	for (auto y : g[x])
		if (!dfn[y])
			dfs(y), low[x] = min(low[x], low[y]);
		else if (!scc[y])
			low[x] = min(low[x], dfn[y]);
	if (dfn[x] == low[x]) {
		int cur = -1; num++;
		while (cur != x)
			scc[cur = sta[tp --]] = num;
	}
}

int opp[maxm], du[maxm], vis[maxm], col[maxn];
vector<int> G[maxm];

void gao() {
	for (int i = 0; i < N; i += 1) if (!dfn[i]) dfs(i);
	for (int i = 0; i < n; i += 1) if (scc[i << 1] == scc[i << 1 | 1]) return void(puts("NO"));
	//
	for (int i = 0; i < n; i += 1) 
		opp[scc[i << 1]] = scc[i << 1 | 1],
		opp[scc[i << 1 | 1]] = scc[i << 1];
	
	for (int x = 0; x < N; x += 1)
		for (auto y : g[x])
			if (scc[x] != scc[y])
				G[scc[y]].pb(scc[x]), du[scc[x]]++;
	
	queue<int> q;
	for (int i = 1; i <= num; i += 1) if (!du[i]) q.push(i);
	while (!q.empty()) {
		int x = q.front(); q.pop();
		if (!vis[x])
			vis[x] = 1, vis[opp[x]] = -1;
		for (auto y : G[x])
			if (!--du[y]) q.push(y);
	}
	for (int i = 0; i < (n << 1); i += 1) if (~vis[scc[i]]) col[i >> 1] = i & 1;
	
	puts("YES");
	for (int i = 0; i < n; i += 1) {
		for (int j = sst[i]; j < sed[i]; j += 1) {
			if (outs[j] == '?') printf("%d", col[i]);
			else printf("%c", outs[j]);
		}
		puts("");
	}
}

void outg() {
	for (int i = 0; i < N; i += 1) {
		for (int j = 0; j < SZ(g[i]); j += 1)
		printf("%d->%d\n", i, g[i][j]);
	}
	puts("");
}

int main()
{
	freopen("binary.in", "r", stdin);
	freopen("binary.out", "w", stdout);

	scanf("%d", &n);
	for (int i = 0; i < n; i += 1) {
		scanf("%s", s);
		m = strlen(s);
		sst[i] = sn;
		for (int j = 0; j < m; j += 1) outs[sn++] = s[j];
		sed[i] = sn;
		for (int j = 0; j < m; j += 1) {
			if (s[j] == '?') {
				s[j] = '0'; add(i << 1);
				s[j] = '1'; add(i << 1 | 1);
				break;
			}
			if (j == m - 1)
				add(i << 1), add(i << 1 | 1);
		}
	}
	generate(); //outg();
	gao();
	return 0;
}
```