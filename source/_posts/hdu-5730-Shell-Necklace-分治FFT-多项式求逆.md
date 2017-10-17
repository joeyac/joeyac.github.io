---
title: 'hdu 5730 Shell Necklace - [分治FFT|多项式求逆]'
categories:
  - ACM
tags:
  - CDQ分治
  - 多项式求逆
  - FFT
date: 2017-10-17 13:48:43
---

给出长度分别为 $ 1 \sim n $ 的珠子，长度为i的珠子有$a[i]$种，每种珠子有无限个，问用这些珠子串成长度为n的链有多少种方案。

<!--more-->

## 链接
[Shell Necklace](http://acm.split.hdu.edu.cn/showproblem.php?pid=5730)

## 题解
( 多么好的一个模板题！业界良心！

令dp[i]表示用这些珠子串成长度为i的链的方案数，并令dp[0]=1，轻易得到转移方程 

$dp[i]=dp[i-1]*a[1]+dp[i-2]*a[2]+...+dp[1]*a[i-1]+dp[0]*a[i]$

即 $dp[i]=\sum_{j=0}^{i-1}dp[j]*a[i-j]$

### 解法一

对于这个式子一个显然的套路就是CDQ分治+FFT

现在考虑用cdq分治处理区间$[l,r]$,按照常规，$mid=(l+r)/2,$我们可以先递归处理$[l,mid]$区间，

然后将$[l,mid]$区间对$[mid+1,r]$区间的影响累加上去，

现在假设$cdq(l,mid)$函数已经求出了$dp[l],dp[l+1]...dp[mid]$的值，

我们考虑如何将其对后半段区间的影响累加上去：

设$g[i]$表示$dp[l],dp[l+1]...dp[mid]$对$dp[i]$的影响，

显然由最开始的DP转移式子，我们可以得出

$g[i]=dp[l]*a[i-l]+dp[l+1]*a[i-l-1]+...=\sum_{j=l}^{mid}dp[j]*a[i-j]$

那么对于区间$[l,mid]$和$[mid+1,r]$,

$dp[l] \ \ dp[l+1] \ \ ... \ \ dp[mid] $

$a[1] \ \ a[2] \ \ a[3] \ \ ... \ \ a[mid+1-l]$ 

把以上两个式子进行卷积即可，然后把对应的影响累加到$dp[mid+1],dp[mid+2]...dp[r]$上，

再用cdq处理右半部分区间即可。

注意不要忘了l==r时的终止条件。

### 解法二

对 $f_n=\sum_{i=0}^{n-1}f_i*a_{n - i}$ 考虑生成函数。

设 $A(x) = a_0x^0 + a_1x^1 + a_2x^2 + ...$
   
设 $F(x) = f_0x^0 + f_1x^1 + f_2x^2 + ...$

显然
$$
\begin{align}
A(x) * F(x) &= 0 + a_1x^1 + (a_1f_1 + a_2f_0)x^2 + ...\\
 &= F(x) - 1
\end{align}
$$

即
$$F(x) = \dfrac{1}{1 - A(x)} $$

那么直接用FFT多项式求逆即可，由于模数很小仅为 $313$

理论上可以用double直接怼过去（~~我没试过啊口胡的~~　CDQ分治＋FFT就是直接double怼过去的）

比较稳妥的做法是用NTT进行多项式求逆，但是并不是一个很舒服的做法

因为 $313$ 并不是一个特殊的素数，常规做法需要CRT进行合并

但是我们现在有了新的黑科技！叫做拆系数FFT(也称MTT)，可以处理任意模数FFT问题

[MTT学习链接](http://blog.csdn.net/samjia2000/article/details/65661468)

同时这个博客中还提到了一种FFT卡常数技巧……

## 代码
### 解法一

CDQ分治 + FFT

```cpp
/*
* Filename:    hdu5730.cpp
* Created:     Tuesday, October 17, 2017 12:57:58 PM
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

const int maxn = 1e5 + 7;
const int maxLen = 18, maxm = 1 << maxLen | 1;
const ll maxv = 1e10 + 6; // 1e14, 1e15
const DB pi = acos(-1.0); // double is enough
ll mod = 313, nlim, sp, msk;

#define _ %mod
#define __ %=mod

ll qpow(ll x, ll p) {
	ll ret = 1;
	while (p) {
		if (p & 1) (ret *= x) __;
		(x *= x) __;
		p >>= 1;
	}
	return ret;
}

namespace FFT{
	struct cp {
		DB r, i;
		cp() {}
		cp(DB r, DB i) : r(r), i(i) {}
		cp operator + (cp const &t) const { return cp(r + t.r, i + t.i); }
		cp operator - (cp const &t) const { return cp(r - t.r, i - t.i); }
		cp operator * (cp const &t) const { return cp(r * t.r - i * t.i, r * t.i + i * t.r); }
		cp conj() const { return cp(r, -i); }
	} w[maxm], wInv[maxm];

	void init() {
		for(int i = 0, ilim = 1 << maxLen; i < ilim; ++i) {
		    int j = i, k = ilim >> 1; // 2 pi / ilim
		    for( ; !(j & 1) && !(k & 1); j >>= 1, k >>= 1);
		    w[i] = cp(cos(pi / k * j), sin(pi / k * j));
		    wInv[i] = w[i].conj();
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
	
	void polyMul(int a[], int aLen, int b[], int bLen, int c[]) { // c not in {a, b}
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
		for(int i = 0; i < cLen; ++i) {
		    int v11 = (ll)(C[i].r + 0.5) % mod, v12 = (ll)(C[i].i + 0.5) % mod;
		    int v21 = (ll)(D[i].r + 0.5) % mod, v22 = (ll)(D[i].i + 0.5) % mod;
		    c[i] = (((((ll)v22 << sp) + v12 + v21) << sp) + v11) % mod;
		}
	}
};

int n, ar[maxn], dp[maxn], a[maxn], b[maxn], c[maxm];
void cdq(int l,int r){
    if(l==r){
        (dp[l]+=ar[l])__;
        return;
    }
    int mid=(l+r)>>1;
    cdq(l,mid);
    for (int i = l; i <= mid; i += 1)
        b[i-l]=dp[i];
    for (int i = 0; i <= r-l+1; i += 1)
        a[i]=ar[i];
    FFT::polyMul(a,r-l+1+1,b,mid-l+1,c);
    for (int i = mid+1; i <= r; i += 1)
        (dp[i]+=ll(c[i-l-1]+0.5))__;
    cdq(mid+1,r);
}

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	//freopen("data.out", "w", stdout);
#endif
	FFT::init();
	a[0] = 1;
	while (scanf("%d", &n) && n) {
		for (int i = 0; i < n; i += 1) scanf("%d", ar + i), ar[i] __, dp[i] = 0;
		cdq(0, n - 1);
		printf("%d\n", dp[n - 1]);
	}
	return 0;
}
```


## 解法二

多项式求逆

```cpp
/*
* Filename:    hdu5730.cpp
* Created:     Tuesday, October 17, 2017 12:57:58 PM
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

const int maxn = 1e5 + 7;
const int maxLen = 18, maxm = 1 << maxLen | 1;
const ll maxv = 1e10 + 6; // 1e14, 1e15
const DB pi = acos(-1.0); // double is enough
ll mod = 313, nlim, sp, msk;

#define _ %mod
#define __ %=mod

ll qpow(ll x, ll p) {
	ll ret = 1;
	while (p) {
		if (p & 1) (ret *= x) __;
		(x *= x) __;
		p >>= 1;
	}
	return ret;
}

namespace FFT{
	struct cp {
		DB r, i;
		cp() {}
		cp(DB r, DB i) : r(r), i(i) {}
		cp operator + (cp const &t) const { return cp(r + t.r, i + t.i); }
		cp operator - (cp const &t) const { return cp(r - t.r, i - t.i); }
		cp operator * (cp const &t) const { return cp(r * t.r - i * t.i, r * t.i + i * t.r); }
		cp conj() const { return cp(r, -i); }
	} w[maxm], wInv[maxm];

	void init() {
		for(int i = 0, ilim = 1 << maxLen; i < ilim; ++i) {
		    int j = i, k = ilim >> 1; // 2 pi / ilim
		    for( ; !(j & 1) && !(k & 1); j >>= 1, k >>= 1);
		    w[i] = cp(cos(pi / k * j), sin(pi / k * j));
		    wInv[i] = w[i].conj();
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
	
	void polyMul(int a[], int aLen, int b[], int bLen, int c[]) { // c not in {a, b}
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
		for(int i = 0; i < cLen; ++i) {
		    int v11 = (ll)(C[i].r + 0.5) % mod, v12 = (ll)(C[i].i + 0.5) % mod;
		    int v21 = (ll)(D[i].r + 0.5) % mod, v22 = (ll)(D[i].i + 0.5) % mod;
		    c[i] = (((((ll)v22 << sp) + v12 + v21) << sp) + v11) % mod;
		}
	}
	
	int c[maxm], tmp[maxm];
	// y should clear to 0
	void polyInv(int x[], int y[], int deg) {
		if (deg == 1) {
			y[0] = qpow(x[0], mod - 2);
			return;
		}
		polyInv(x, y, (deg + 1) >> 1);
	
		copy(x, x + deg, tmp);
		int p = ((deg + 1) >> 1) + deg - 1;
		polyMul(y, (deg + 1) >> 1, tmp, deg, c);
	
		for (int i = 0; i < p; i += 1) c[i] = (- c[i] + mod) _;
		(c[0] += 2) __;
	
		polyMul(y, (deg + 1) >> 1, c, deg, tmp);
		copy(tmp, tmp + deg, y);
	}
	
};

int n, a[maxn], b[maxn];

int main()
{
#ifdef AC
	freopen("data.in", "r", stdin);
	//freopen("data.out", "w", stdout);
#endif
	FFT::init();
	a[0] = 1;
	while (scanf("%d", &n) && n) {
		for (int i = 0; i <= n; i += 1) b[i] = 0;
		for (int i = 1; i <= n; i += 1) scanf("%d", a + i), a[i] = mod - a[i] _;
		FFT::polyInv(a, b, n + 1);
		printf("%d\n", b[n]);
	}
	return 0;
}
```