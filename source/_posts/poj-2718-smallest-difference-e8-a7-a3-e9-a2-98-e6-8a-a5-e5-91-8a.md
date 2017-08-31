---
title: poj 2718 Smallest Difference 解题报告
id: 173
categories:
  - ACM
date: 2016-03-08 13:45:33
tags:
  - 题解
---

链接:[poj2718](http://poj.org/problem?id=2718)
> <div class="ptt" lang="en-US">Smallest Difference</div>
> 
> <div class="plm">
> 
> <table align="center">
> 
> <tbody>
> 
> <tr>
> 
> <td>**Time Limit:** 1000MS</td>
> 
> <td>**Memory Limit:** 65536K</td>
> 
> </tr>
> 
> <tr>
> 
> <td>**Total Submissions:** 7259</td>
> 
> <td>**Accepted:** 1974</td>
> 
> </tr>
> 
> </tbody>
> 
> </table>
> 
> </div>
> 
> 
> Description
> 
> <div class="ptx" lang="en-US">
> 
> 
> Given a number of distinct decimal digits, you can form one integer by choosing a non-empty subset of these digits and writing them in some order. The remaining digits can be written down in some order to form a second integer. Unless the resulting integer is 0, the integer may not start with the digit 0.
> 
> 
> For example, if you are given the digits 0, 1, 2, 4, 6 and 7, you can write the pair of integers 10 and 2467\. Of course, there are many ways to form such pairs of integers: 210 and 764, 204 and 176, etc. The absolute value of the difference between the integers in the last pair is 28, and it turns out that no other pair formed by the rules above can achieve a smaller difference.
> 
> 
> </div>
> 
> 
> Input
> 
> <div class="ptx" lang="en-US">The first line of input contains the number of cases to follow. For each case, there is one line of input containing at least two but no more than 10 decimal digits. (The decimal digits are 0, 1, ..., 9.) No digit appears more than once in one line of the input. The digits will appear in increasing order, separated by exactly one blank space.</div>
> 
> 
> Output
> 
> <div class="ptx" lang="en-US">For each test case, write on a single line the smallest absolute difference of two integers that can be written from the given digits as described by the rules above.</div>
> 
> 
> Sample Input
> 
> <pre class="sio">1
> 
> 0 1 2 4 6 7
> 
> </pre>
> 
> 
> Sample Output
> 
> <pre class="sio">28
> 
> </pre>
> 
> 
> Source
> 
> <div class="ptx" lang="en-US">[Rocky Mountain 2005](http://poj.org/searchproblem?field=source&amp;key=Rocky+Mountain+2005)</div>
<div class="ptx" lang="en-US">一道题暴露了自己在c++方面字符串处理的薄弱,还有输入输出,在这里顺便先做个总结吧:</div>
<div class="ptx" lang="en-US">1.首先是所谓io的黑魔法一句话(关闭cin,cout缓冲区,慎用):</div>
<div class="ptx" lang="en-US">
<pre class="lang:c++ decode:true ">ios_base::sync_with_stdio(0);

std::ios::sync_with_stdio(false);</pre>
</div>
<div class="ptx" lang="en-US">

2.输入输出重定向:(最后两句可以不加)
<pre class="lang:c++ decode:true">freopen("date.in","r",stdin);  //重定向所有标准的输入为文件输入

freopen("date.out","w",stdout);//重定向所有标准的输出为文件输出

fclose(stdin);
fclose(stdout);//输出结束</pre>
下面是针对online judge 的处理:
<pre class="lang:c++ decode:true">#ifndef ONLINE_JUDGE 
    freopen("in.txt","r",stdin); 
    freopen("out.txt","w",stdout); 
#endif 

#ifndef ONLINE_JUDGE 
    fclose(stdin); 
    fclose(stdout); 
#endif</pre>
</div>
其实还可以在自己的编译参数内预先加上定义,比如
<pre class="lang:c++ decode:true ">g++ $fullname -o $name -DIamDefine -Wall -std=gnu++0x -static -lm</pre>
然后,
<pre class="lang:c++ decode:true">#ifdef IamDefine
	freopen("data.in","r",stdin);
	freopen("data.out","w",stdout);
#endif</pre>
&nbsp;

3.读入整行(去除空格):
<pre class="lang:c++ decode:true">string line;
getline(cin,line);
line.erase(remove(line.begin(),line.end(),' '),line.end());</pre>
<pre class="lang:c++ decode:true ">while ((ch=getchar())!='\n')
{
	if(ch==' ')continue;
	a[l++]=ch;
}</pre>
4.字符串,字符串数组和数字转换处理:
<pre class="lang:c++ decode:true">char a[10];
string temp=string(a);
int x=atoi(temp.substr(0,half).c_str());</pre>
另外还有atoi(),atol(),atof().等等.

最后提一点,判断字符时记得加  ' ' !!!  char a[n],   if(a[n]==1)   ???    excuse me???

AC代码:
<pre class="lang:c++ decode:true">/*
* Filename:    poj2718.cpp
* Desciption:  穷竭搜索
* Created:     2016-03-07
*
*/
#include&lt;iostream&gt;
#include&lt;stdio.h&gt;
#include&lt;algorithm&gt;
#include&lt;cstring&gt;
#include&lt;map&gt;
#define INT_MAX 1&lt;&lt;30
using namespace std;
//typedef long long ll;
const int INF=0x7F;
int a[10];
int l,ans;
int getnum(int s,int e){
	int res=0;
	for (int i = s; i &lt; e; i += 1)
	{
		res+=a[i];
		if(i!=e-1)res*=10;
	}
	return res;
}
void solve(){
	int half=l/2;
	int x,y;
	do
	{
		if((a[0]!=0||half==1)&amp;&amp;(a[half]!=0||l-half==1)){
			x=getnum(0,half);
			y=getnum(half,l);
			ans=min(ans,abs(x-y));
		}
	} while (next_permutation(a,a+l));

}
int main()
{
#ifdef FUCK_PROBLEM
	freopen("data.in","r",stdin);
	freopen("data.out","w",stdout);
#endif
	int t;
	scanf("%d ",&amp;t);
	while (t--)
	{
		l=0;
		ans=100000;
		char ch;
		while ((ch=getchar())!='\n')
		{
			if(ch==' ')continue;
			a[l++]=ch-'0';
		}
		solve();
		printf("%d\n",ans);
	}
	return 0;
}</pre>
&nbsp;

&nbsp;