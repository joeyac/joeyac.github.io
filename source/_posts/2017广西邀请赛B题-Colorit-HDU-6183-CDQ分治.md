---
title: 2017广西邀请赛B题-Colorit-HDU-6183-CDQ分治
date: 2017-09-07 21:15:12
categories:
  - ACM
tags:
  - CDQ分治
  - 线段树
---

维护两种操作，操作1往点$(x, y) (1 \leq x, y \leq 10^6)$加入一个颜色为$c (0 \leq c \leq 50)$的点，操作2查询矩形区域 $(1 \leq a \leq x, y_1 \leq b \leq y_2) (1 \leq x, y_1, y_2 \leq 10^6)$区域中所有$(a, b)$点的颜色种类数。

<!--more-->

## 链接
[Color it](http://acm.hdu.edu.cn/showproblem.php?pid=6183)

## 题解
首先要注意到颜色最多只有51种，51这个数字足够进行状态压缩，或者做一些其他的trick，然后对于类似的区间查询题目，因为大多数满足区间减法的性质，查询上可能会随便给，这个题不一样，有一个很明显的特殊的地方就是，矩形的左边是从1开始的，然后注意到题目中的输入格式，可以理解为多组输入（最多10组），然后每组操作数最多$1.5 × 10^5 $，那么就可以考虑对每组操作进行离线处理，考察修改操作对查询操作的贡献。

然后注意到题目中其实有一个三维偏序关系，即操作顺序（时间），x坐标，y坐标，很自然的，可以想到CDQ分治是否能维护，首先第一维按时间顺序，第二维x坐标用CDQ维护，第三维y坐标用数据结构维护，当$t_i < t_j \ and \ x_i <= x_j $ 且$i$是修改操作，$j$是查询操作时，由于矩形查询左端一定接到$x=1$, 所以就只剩下一个第三维$y$坐标需要维护了，修改和查询分别相当于单点修改和区间查询，具体可以将颜色用$long  \ long$压位，然后用线段树维护，每次两个值直接取或即可。

### Something About CDQ
关于CDQ具体细节，这题可以是第一维时间也可以是第一维x坐标，不过y坐标是肯定要作为第三维的，两种做法并无多大区别，时间复杂度上是一样的，只是具体实现上和思路上有那么一点区别，具体看代码。

需要注意的一点是，需要时刻记住CDQ过程中需要保证在第一维和第二维都是不大于的关系时才可以去更新第三维……自己赛时犯蠢……CDQ过程中把区间分成两半之后才进行数据结构维护……其实应该一边分一边维护……大概是意识模糊了……

### UPDATE
题解还有开51棵线段树分别维护各个点的搞法……回头有空再看

## 代码
以时间作为第一维，X坐标作为第二维，不需要对操作提前进行排序，cdq函数需要四个参数。
```cpp
/*
* Filename:    B.cpp
* Created:     Thursday, September 07, 2017 02:30:58 PM
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
const int maxn = 300000 + 7;
const int N = 1e6 + 7;

int n, tot, op;

ll ans[N];

struct SegTree{
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
#define arg int l=1,int r=N,int rt=1
#define args l,r,rt
    ll sum[4*N];
    bool Clr[4*N];
    void build(){
        clr(sum,0); clr(Clr,0);
    }
    void clean(){Clr[1]=true;sum[1]=0;}
    void pu(int rt){sum[rt]=sum[rt<<1] | sum[rt<<1|1];}
    void pd(int l,int r,int rt){
        if(Clr[rt]){
            sum[rt<<1]=sum[rt<<1|1]=0;
            Clr[rt<<1]=Clr[rt<<1|1]=true;
            Clr[rt]=false;
        }
    }
    void update(int x,ll v,arg){
        if(l == r)sum[rt] |= v;
        else{
            pd(args);
            int m=(l+r)>>1;
            if(x<=m)update(x,v,lson);
            else update(x,v,rson);
            pu(rt);
        }
    }
    ll query(int L,int R, arg){
        if(L<=l&&r<=R)return sum[rt];
        else{
            pd(args);
            int m=(l+r)>>1;
            ll ret = 0;
            if(L<=m)ret |= query(L,R,lson);
            if(R>m)ret |= query(L,R,rson);
            pu(rt);
            return ret;
        }
    }
}S;

struct oper{
	int id, type;
	int x, y, v;
	void in(int i, int t) {
		scanf("%d%d%d", &x, &y, &v);
		id = i; 
		type = t;
	}
}Q[maxn], Q1[maxn], Q2[maxn];

void cdq(int l, int r, int ql, int qr) {
	if (ql == qr) return;
	if (l == r) {
		S.clean();
		for (int i = ql; i <= qr; i += 1) {
			if (Q[i].type == 1) S.update(Q[i].y, 1ll<<Q[i].v);
			else ans[Q[i].id] |= S.query(Q[i].y, Q[i].v);
		}
		return;
	}
	
	int mid = (l + r) >> 1;
	
	int c1 = 0, c2 = 0;
	S.clean();
	for (int i = ql; i <= qr; i += 1) {
		if (Q[i].x <= mid) {
			Q1[c1++] = Q[i];
			if (Q[i].type == 1) S.update(Q[i].y, 1ll<<Q[i].v);
		} else {
			Q2[c2++] = Q[i];
			if (Q[i].type == 2) ans[Q[i].id] |= S.query(Q[i].y, Q[i].v);
		}
	}
	for (int i = 0; i < c1; i += 1)Q[i + ql] = Q1[i];
	for (int i = 0; i < c2; i += 1)Q[i + ql + c1] = Q2[i];
	
	cdq(l, mid, ql, ql + c1 - 1);
	cdq(mid + 1, r, ql + c1, qr);
}

int gao(ll x) {
	int ret  = 0;
	while (x) {
		if (x & 1) ret++;
		x>>=1;
	}
	return ret;
}

void sol() {
	if (n == 0) return;
	for (int i = 0; i < n; i += 1) ans[i] = Q[i].type - 2;
	cdq(1, 1e6, 0, n - 1);
	for (int i = 0; i < n; i += 1) {
		if (ans[i] == -1) continue;
		printf("%d\n", gao( ans[i] ) );
	}
	n = 0;
}

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	freopen("B3.out", "w", stdout);
#endif
	S.build();
	n = 0;
	while ( scanf("%d",&op) ) {
		if (op == 1 || op == 2) Q[n].in(n, op), n++;
		else if (op == 0) sol();
		else {sol(); break;}
	}
	return 0;
}
```

以X坐标作为第一维，时间作为第二维，需要预先对查询排序,但是cdq函数只需要两个参数：
```cpp
/*
* Filename:    B.cpp
* Created:     Thursday, September 07, 2017 02:30:58 PM
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
const int maxn = 300000 + 7;
const int N = 1e6 + 20;

int n, tot, op;

ll ans[N];

struct SegTree{
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
#define arg int l=1,int r=N,int rt=1
#define args l,r,rt
    ll sum[4*N];
    bool Clr[4*N];
    void build(){
        clr(sum,0); clr(Clr,0);
    }
    void clean(){Clr[1]=true;sum[1]=0;}
    void pu(int rt){sum[rt]=sum[rt<<1] | sum[rt<<1|1];}
    void pd(int l,int r,int rt){
        if(Clr[rt]){
            sum[rt<<1]=sum[rt<<1|1]=0;
            Clr[rt<<1]=Clr[rt<<1|1]=true;
            Clr[rt]=false;
        }
    }
    void update(int x,ll v,arg){
        if(l == r)sum[rt] |= v;
        else{
            pd(args);
            int m=(l+r)>>1;
            if(x<=m)update(x,v,lson);
            else update(x,v,rson);
            pu(rt);
        }
    }
    ll query(int L,int R, arg){
        if(L<=l&&r<=R)return sum[rt];
        else{
            pd(args);
            int m=(l+r)>>1;
            ll ret = 0;
            if(L<=m)ret |= query(L,R,lson);
            if(R>m)ret |= query(L,R,rson);
            pu(rt);
            return ret;
        }
    }
}S;

struct oper{
	int id, type;
	int x, y, v;
	void in(int i, int t) {
		scanf("%d%d%d", &x, &y, &v);
		id = i; 
		type = t;
	}
}Q[maxn], Q1[maxn], Q2[maxn];

bool cmp(const oper &x1, const oper &x2){
	if (x1.x == x2.x) return x1.type < x2.type; 
	return x1.x < x2.x;
}

void cdq(int l, int r) {
	if (l == r)  return;
	
	int mid = (l + r) >> 1;
	
	int c1 = 0, c2 = 0;
	S.clean();
	for (int i = l; i <= r; i += 1) {
		if (Q[i].id <= mid) {
			Q1[c1++] = Q[i];
			if (Q[i].type == 1) S.update(Q[i].y, 1ll<<Q[i].v);
		} else {
			Q2[c2++] = Q[i];
			if (Q[i].type == 2) ans[Q[i].id] |= S.query(Q[i].y, Q[i].v);
		}
	}
	for (int i = 0; i < c1; i += 1)Q[i + l] = Q1[i];
	for (int i = 0; i < c2; i += 1)Q[i + l + c1] = Q2[i];

	cdq(l, mid);
	cdq(mid + 1, r);
}

int gao(ll x) {
	int ret  = 0;
	while (x) {
		if (x & 1) ret++;
		x>>=1;
	}
	return ret;
}

void sol() {
	if (n == 0) return;
	for (int i = 0; i < n; i += 1) ans[i] = Q[i].type - 2;
	sort(Q, Q + n, cmp);
	cdq(0, n - 1);
	for (int i = 0; i < n; i += 1) {
		if (ans[i] == -1) continue;
		printf("%d\n", gao(ans[i]) );
	}
	n = 0;
}

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	freopen("B4.out", "w", stdout);
#endif
	S.build();
	n = 0;
	while ( scanf("%d",&op) ) {
		if (op == 1 || op == 2) Q[n].in(n, op), n++;
		else if (op == 0) sol();
		else {sol(); break;}
	}
	return 0;
}
```