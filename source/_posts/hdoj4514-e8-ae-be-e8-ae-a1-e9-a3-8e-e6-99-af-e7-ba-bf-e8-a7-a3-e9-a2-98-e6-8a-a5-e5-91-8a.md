---
title: hdoj4514 设计风景线 解题报告
tags:
  - 图论
  - 并查集
  - 树
  - 树的直径
id: 132
categories:
  - ACM
date: 2016-02-19 21:34:31
---

链接：[hdoj4514](http://acm.hdu.edu.cn/showproblem.php?pid=4514)
> # 湫湫系列故事——设计风景线
> 
> **Time Limit: 6000/3000 MS (Java/Others)    Memory Limit: 65535/32768 K (Java/Others)
> 
> Total Submission(s): 3730    Accepted Submission(s): 654
> 
> **
> 
> <div class="panel_title" align="left">Problem Description</div>
> 
> <div class="panel_content">　　随着杭州西湖的知名度的进一步提升，园林规划专家湫湫希望设计出一条新的经典观光线路，根据老板马小腾的指示，新的风景线最好能建成环形，如果没有条件建成环形，那就建的越长越好。
> 
> 现在已经勘探确定了n个位置可以用来建设，在它们之间也勘探确定了m条可以设计的路线以及他们的长度。请问是否能够建成环形的风景线？如果不能，风景线最长能够达到多少？
> 
> 其中，可以兴建的路线均是双向的，他们之间的长度均大于0。</div>
> 
> &nbsp;
> 
> <div class="panel_title" align="left">Input</div>
> 
> <div class="panel_content">　　测试数据有多组，每组测试数据的第一行有两个数字n, m，其含义参见题目描述；
> 
> 接下去m行，每行3个数字u v w，分别代表这条线路的起点，终点和长度。**[Technical Specification]**
> 
> 1\. n&lt;=100000
> 
> 2\. m &lt;= 1000000
> 
> 3\. 1&lt;= u, v &lt;= n
> 
> 4\. w &lt;= 1000
> 
> 
> </div>
> 
> &nbsp;
> 
> <div class="panel_title" align="left">Output</div>
> 
> <div class="panel_content">　　对于每组测试数据，如果能够建成环形（并不需要连接上去全部的风景点），那么输出YES，否则输出最长的长度，每组数据输出一行。</div>
> 
> &nbsp;
> 
> <div class="panel_title" align="left">Sample Input</div>
> 
> <div class="panel_content">
> 
> <div>3 3 1 2 1 2 3 1 3 1 1</div>
> 
> </div>
> 
> &nbsp;
> 
> <div class="panel_title" align="left">Sample Output</div>
> 
> <div class="panel_content">
> 
> <div>YES</div>
> 
> </div>
<div class="panel_content">
<div>整整一个晚上加一个下午还没解决的题Orz....</div>
<div>整整提交了20次都是MLE的题Orz.....</div>
<div>判断是否能形成环形的路线用并查集就可以,这部分处理起来不难;</div>
<div>至于求最长路,一开始是想着修改**Dijkstra**算法并利用标记数组,仔细想想,因为是求最长路径,这样的贪心算法是不正确的.</div>
<div>于是在查找资料的时候学习了SPFA算法,(已经添加了相关介绍在本blog的[图论总结](http://jingwei.site/map-algorithm/)中),输入时记录每个点的出度,(由于是无向图每次将两个点的出度都加1),然后如果判出不含环,遍历所有出度为1的点,分别以它们为起点进行SPFA,这个思路是没错的,时间也是够的,但是会造成MLE....至今不知为何,,,,</div>
<div></div>
<div>最后还是在学姐的指导下了解到这是求树的直径....之前完全没有接触过嘛...</div>
> <div>树的直径是指树的最长简单路。求法: 两遍BFS :先任选一个起点BFS找到最长路的终点，再从终点进行BFS，则第二次BFS找到的最长路即为树的直径；
> 
> 原理: 设起点为u,第一次BFS找到的终点v一定是树的直径的一个端点
> 
> 证明: 1) 如果u 是直径上的点，则v显然是直径的终点(因为如果v不是的话，则必定存在另一个点w使得u到w的距离更长，则于BFS找到了v矛盾)
> 
> 2) 如果u不是直径上的点，则u到v必然于树的直径相交(反证),那么交点到v 必然就是直径的后半段了
> 
> 所以v一定是直径的一个端点，所以从v进行BFS得到的一定是直径长度</div>
<div>但是这题还需要注意一个问题,可能存在多个联通分量,需要分别求出最长路.</div>
<div>然而,依旧MLE....%&gt;_&lt;%....坐等更新:</div>
<div>和学姐折腾了一整个晚上,终于是解决了,思路就是并查集判环和求树的直径,但是由于题目对内存限制相当严格,g++提交到杭电一直MLE...最后用C++提交就过了...按照学姐的话来说,杭电的g++版本有点老,,,,g++和c++对vector数组的底层实现不同,我的代码应该只超了一点点内存.....Orz.....</div>
<div>下面是AC代码:</div>
<div>
<pre class="lang:c++ decode:true ">#include&lt;iostream&gt;
#include&lt;memory.h&gt;
#include&lt;stdio.h&gt;
#include &lt;vector&gt;
#include&lt;queue&gt;
using namespace std;
const int max_N=100000+7;
int f[max_N],rk[max_N];
struct edge
{
	int to,len;
};
edge e1,e2;
vector&lt;edge&gt; G[max_N];
void init(int n){
	for (int i = 0; i &lt; n; i += 1)
	{
		f[i]=i;
		rk[i]=0;
	}
}
int find(int x){
	int r=x;	
	while(f[r]!=r){
		r=f[r];
	}
	int t=x,j;
	while (t!=r)
	{
		j=f[t];
		f[j]=r;
		t=j;
	}
	return r;
}
void unite(int x,int y){
	x=find(x);
	y=find(y);
	if(x==y)return;
	if(rk[x]&gt;rk[y]){
		f[y]=x;
	}else{
		f[x]=y;
		if(rk[x]==rk[y])rk[y]++;
	}
}
bool same(int x,int y){
	return find(x)==find(y);
}
int n,m,a,b,c;
bool bl=false;

int ans=0,d[max_N],maxd;
bool vis[max_N],visal[max_N];
queue&lt;int&gt; que;
int bfs(int sp){   
	que.push(sp);
	memset(vis,false,n*sizeof(bool));
	memset(d,0,n*sizeof(bool));
	maxd=0;
	int an=sp;
	vis[sp]=true;visal[sp]=true;d[sp]=0;
	while (!que.empty())
	{
		int x=que.front();que.pop();
		for (unsigned int i = 0; i &lt; G[x].size(); i += 1)
		{
			edge s=G[x][i];
			int y=s.to,c=s.len;
			if(!vis[y]){
				que.push(y);
				vis[y]=visal[y]=true;
				d[y]=d[x]+c;
				if(maxd&lt;d[y]){
					maxd=d[y];
					an=y;
				}
			}
		}
	}
	return an;
}
int main()
{
	while (scanf("%d%d",&amp;n,&amp;m)!=EOF)
	{
		bl=false;
		init(n);
		for (int i = 0; i &lt; max_N; i += 1)G[i].clear();
		for (int i = 0; i &lt; m; i += 1)
		{
			scanf("%d%d%d",&amp;a,&amp;b,&amp;c);
			a=a-1;
			b=b-1;
			e1.to=a;e1.len=c;
			e2.to=b;e2.len=c;
			G[a].push_back(e2);
			G[b].push_back(e1);
			if(!bl){
				if(same(a,b))bl=true;
				else unite(a,b);
			}
		}
		if(bl)printf("YES\n");
		else{
			ans=0;
			memset(visal,0,sizeof(bool)*n);
			for (int i = 0; i &lt; n; i += 1)
			{
				if(!visal[i]){
					int s=bfs(i);
					s=bfs(s);
					if(maxd&gt;ans)ans=maxd;
				}
			}
			printf("%d\n",ans);
		}
	}
	return 0;
}
</pre>
&nbsp;

</div>
</div>