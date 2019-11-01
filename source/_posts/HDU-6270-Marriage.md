---
title: HDU 6270 - Marriage
mathjax: true
categories:
  - ACM
tags:
  - FFT
date: 2018-05-14 17:56:23
---

给出$n$个家庭，每个家庭有$a_i$个男生，$b_i$个女生，保证$\sum{a_i}=\sum{b_i} \leq 10^5$, 现在求每个人不会在家庭内部匹配异性的方案数对$998244353$取模。

<!--more-->
## 链接
[Marriage](http://acm.split.hdu.edu.cn/showproblem.php?pid=6270)

注意题目的pdf在[这儿](http://acm.hdu.edu.cn/downloads/CCPC2018-Hangzhou-ProblemSet.pdf)……

## 题解
哎赛时并没有做出来，只是感觉像是个卷积的题……而且赛时也没有注意到总人数也限制了……

我们需要求解的问题是合法的匹配数量，考虑一下对立的问题，不合法的情况是什么样的呢？就是有人匹配上了自己家庭内部的异性，如果枚举不合法的匹配个数，且知道每种个数对应的方案$g(x)$，那么简单容斥可得答案应该为$\sum_{i=0}^{m}(-1)^i g(i) (S-i)!$, 其中$m$为最大的可能的不合法匹配数，应该有$m=\sum min(a_i,b_i)$, S为总人数，即$S = \sum a_i$。

现在可以看看如何求这个$g(x)$, 如果只有一个家庭，显然$g(x)=C_a^xC_b^x x!$, $x <= min(a, b)$

如果有两个家庭，类似的有$g_1(x), g_2(x)$，则$g(x)=\sum_{i=0}^{min(a_1,b_1)}\sum_{j=0}^{min(a_2,b_2)}[i + j == x]g_1(i)g_2(j)$

这个时候就已经很明显了，就是一个卷积的形式，所以只需要对每个家庭$i$预处理出来$g_i(x)$,将其用启发式合并的方式卷积起来即可，这样便得到了最终的$g(x)$

## 代码

```cpp
/*
* Filename:    hdu6270.cpp
* Created:     Monday, May 14, 2018 06:34:26 PM
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
typedef double DB;
typedef pair<int, int> P;

int mod = 998244353, nlim, sp, msk;
const int maxn = (1 << 17) + 7;
const int maxLen = 19, maxm = 1 << maxLen | 1;
const ll maxv = 1e9; // 1e14, 1e15
const DB pi = acos(-1.0); // double is enough

inline int add(int a, int b) { return (a + b) % mod; }
inline int sub(int a, int b) { return (a - b + mod) % mod; }
inline int mul(int a, int b) { return (ll)a * (ll)b % mod; }
inline int exp(int a, int b) {
	int ret = 1;
	for (; b; b >>= 1) {
		if (b & 1) ret = mul(ret, a);
		a = mul(a, a);
	}
	return ret;
}
inline int inv(int x) { return exp(x, mod - 2); }

namespace MTT {
	int I2 = inv(2);
	struct cp {
		DB r, i;
		cp() {}
		cp(DB r, DB i) : r(r), i(i) {}
		cp operator + (cp const &t) const { return cp(r + t.r, i + t.i); }
		cp operator - (cp const &t) const { return cp(r - t.r, i - t.i); }
		cp operator * (cp const &t) const { return cp(r * t.r - i * t.i, r * t.i + i * t.r); }
		cp conj() const { return cp(r, -i); }
	} w[maxm];
	void init() {
		for(int i = 0, ilim = 1 << maxLen; i < ilim; ++i) {
		    int j = i, k = ilim >> 1; // 2 pi / ilim
		    for( ; !(j & 1) && !(k & 1); j >>= 1, k >>= 1);
		    w[i] = cp(cos(pi / k * j), sin(pi / k * j));
		}
		nlim = std::min(maxv / (mod - 1) / (mod - 1), maxn - 1LL);
		for(sp = 1; 1 << (sp << 1) < mod; ++sp);
		msk = (1 << sp) - 1;
	}
	
	void FFT(int n, cp a[], int flag) {
		static int bitLen = 0, bitRev[maxm] = {};
		if(n != (1 << bitLen)) {
		    for(bitLen = 0; 1 << bitLen < n; ++bitLen);
		    for(int i = 1; i < n; ++i)
		        bitRev[i] = (bitRev[i >> 1] >> 1) | ((i & 1) << (bitLen - 1));
		}
		for(int i = 0; i < n; ++i)
		    if(i < bitRev[i])
		        std::swap(a[i], a[bitRev[i]]);
		for(int i = 1, d = 1; d < n; ++i, d <<= 1)
		    for(int j = 0; j < n; j += d << 1)
		        for(int k = 0; k < d; ++k) {
		            cp &AL = a[j + k], &AH = a[j + k + d];
		            cp TP = w[k << (maxLen - i)] * AH;
		            AH = AL - TP, AL = AL + TP;
		        }
		if(flag != -1)
		    return;
		std::reverse(a + 1, a + n);
		for(int i = 0; i < n; ++i) {
		    a[i].r /= n;
		    a[i].i /= n;
		}
	}
	
	void polyMul(vector<int> &a, vector<int> &b, vector<int> &c) { // c not in {a, b}
		int aLen = a.size(), bLen = b.size();
		static cp A[maxm], B[maxm], C[maxm], D[maxm];
		int len, cLen = aLen + bLen - 1; // optional: parameter
		for(len = 1; len < aLen + bLen - 1; len <<= 1);
		if(std::min(aLen, bLen) <= nlim) {
		    for(int i = 0; i < len; ++i)
		        A[i] = cp(i < aLen ? a[i] : 0, i < bLen ? b[i] : 0);
		    FFT(len, A, 1);
		    cp tr(0, -0.25);
		    for(int i = 0, j; i < len; ++i)
		        j = (len - i) & (len - 1), B[i] = (A[i] * A[i] - (A[j] * A[j]).conj()) * tr;
		    FFT(len, B, -1);
		    for(int i = 0; i < cLen; ++i) c[i] = (ll)(B[i].r + 0.5) % mod;
		    return;
		} // if min(aLen, bLen) * mod <= maxv
		for(int i = 0; i < len; ++i) {
		    A[i] = i < aLen ? cp(a[i] & msk, a[i] >> sp) : cp(0, 0);
		    B[i] = i < bLen ? cp(b[i] & msk, b[i] >> sp) : cp(0, 0);
		}
		FFT(len, A, 1), FFT(len, B, 1);
		cp trL(0.5, 0), trH(0, -0.5), tr(0, 1);
		for(int i = 0, j; i < len; ++i) {
		    j = (len - i) & (len - 1);
		    cp AL = (A[i] + A[j].conj()) * trL;
		    cp AH = (A[i] - A[j].conj()) * trH;
		    cp BL = (B[i] + B[j].conj()) * trL;
		    cp BH = (B[i] - B[j].conj()) * trH;
		    C[i] = AL * (BL + BH * tr);
		    D[i] = AH * (BL + BH * tr);
		}
		FFT(len, C, -1), FFT(len, D, -1);
		c.clear();
		c.resize(cLen);
		for(int i = 0; i < cLen; ++i) {
		    int v11 = (ll)(C[i].r + 0.5) % mod, v12 = (ll)(C[i].i + 0.5) % mod;
		    int v21 = (ll)(D[i].r + 0.5) % mod, v22 = (ll)(D[i].i + 0.5) % mod;
		    c[i] = (((((ll)v22 << sp) + v12 + v21) << sp) + v11) % mod;
		}
	}
};

int fac[maxn], invfac[maxn];

int C(int a, int b) {
	return mul(mul(fac[a], invfac[b]), invfac[a - b]);
}

int n, m;
vector<int> vec[maxn];

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	//freopen("data.out", "w", stdout);
#endif
	int T, a, b;
	fac[0] = fac[1] = 1;
	invfac[0] = invfac[1] = 1;
	for (int i = 2; i < maxn; i++)
		fac[i] = mul(fac[i - 1], i),
		invfac[i] = mul(invfac[i - 1], inv(i));
	scanf("%d", &T);
	MTT::init();
	while (T--) {
		scanf("%d", &n);
		priority_queue <P, vector<P>, greater<P> > que;
		int S = 0;
		for (int i = 0; i < n; i++) {
			vec[i].clear();
			scanf("%d%d", &a, &b);
			S += a;
			for (int j = 0; j <= min(a, b); j++)
				vec[i].pb(mul( mul(C(a, j), C(b, j)), fac[j]) );
			que.push(P(vec[i].size(), i));
		}
		while (que.size() > 1) {
			P p1 = que.top(); que.pop();
			P p2 = que.top(); que.pop();
			MTT::polyMul(vec[p1.se], vec[p2.se], vec[p1.se]);
			que.push(P(vec[p1.se].size(), p1.se));
		}
		int ans = 0;
		int id = que.top().se;
		for (int i = 0; i < (int)(vec[id].size()); i++) {
			int val = mul( mul(vec[id][i], fac[S - i]), 1 - i % 2 * 2);
			ans = add(ans, val);
		}
		printf("%d\n", add(ans, mod));
	}
	return 0;
}
```