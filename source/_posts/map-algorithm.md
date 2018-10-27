---
title: 图算法
tags:
  - 图论
id: 117
categories:
  - ACM
date: 2016-02-17 21:33:31
---

## 基本概念

图由顶点（vertex，node）和边（edge）组成
顶点:图中的数据元素称为顶点.
有向图:有方向的图叫有向图.
无向图:没有方向的图叫无向图.
完全图:有n(n-1)/2条边的无向图称为完全图.
有向完全图:具有n(n-1)条弧的有向图称为有向完全图.
稀疏图:有很少条边或弧的图称为稀疏图,反之称为稠密图.
权:与图的边或弧相关的数叫做权(weight).
DAG：没有圈的有向图.

## 图的表示

1.邻接矩阵
2.邻接表
邻接表的实现：
<pre class="minimize:true lang:c++ decode:true">#include&lt;iostream&gt;
#include&lt;stdio.h&gt;
#include&lt;vector&gt;
using namespace std;
const int MAX_V=10000;
vector&lt;int&gt; G[MAX_V];
//struct edge{      //边上有属性的情况
//	int to,cost;
//};
//vector&lt;edge&gt; G[MAX_V];
int main()
{
	int V,E,a,b;
	scanf("%d%d",&amp;V,&amp;E);
	for (int i = 0; i &lt; E; i += 1)
	{
		scanf("%d%d",&amp;a,&amp;b);
		G[a].push_back(b);
//		如果是无向图，加上下面这条
//		G[b].push_back(a);
	}
//	图的操作
//	。
//	。
//	。
	return 0;
}</pre>

### 二分图搜索：（DFS）

<pre class="minimize:true lang:c++ decode:true">#include&lt;iostream&gt;
#include&lt;stdio.h&gt;
#include&lt;vector&gt;
using namespace std;
const int MAX_V=10000;
vector&lt;int&gt; G[MAX_V];
int color[MAX_V];
bool dfs(int v,int c){
	color[v]=c;
	for (int i = 0; i &lt; G[v].size(); i += 1)
	{
		if(color[G[v][i]]==c)return false;
		if(color[G[v][i]]==0&amp;&amp;!dfs(G[v][i],-c))return false;
	}
	return true;
}
int main()
{
	int V,E,a,b;
	scanf("%d%d",&amp;V,&amp;E);
	for(int i=0;i&lt;V;i++)color[i]=0;  //初始化
	for (int i = 0; i &lt; E; i += 1)
	{
		scanf("%d%d",&amp;a,&amp;b);
		G[a].push_back(b);
		G[b].push_back(a);
	}
	if(!dfs(0,1)){
		printf("Not A Bipartite Graph");
	}
	else{
		printf("YES");
	}
	return 0;
}</pre>

## 单源最短路问题

松弛操作：
<pre class="lang:c++ decode:true">void relax(int from,int to,int cost){   //一般不需要单独编写
    if(d[from]!=INF&amp;&amp;d[to]&gt;d[from]+cost){
        d[to]=d[from]+cost;
    }
}</pre>

### 1.Bellman-Ford算法

适用条件&amp;范围：
a) 单源最短路径(从源点s到其它所有顶点v);
b) 有向图&amp;无向图(无向图可以看作(u,v),(v,u)同属于边集E的有向图);
c) 边权可正可负(如有负权回路输出错误提示);
d) 差分约束系统;
<pre class="minimize:true lang:c++ decode:true ">void short_path(int s){
	for(int i=0;i&lt;V;i++)D[i]=INF;
	D[s]=0;
	while(true){
		bool update=false;
		for (int i = 0; i &lt; E; i += 1)
		{
			edge e=es[i];
			if(D[e.from]!=INF&amp;&amp;D[e.to]&gt;D[e.from]+e.cost){
				D[e.to]=D[e.from]+e.cost;
				update=true;
			}
		}
		if(!update)break;
	}
}</pre>
如果在图中不存在从s可达的负圈，那么最短路不会经过同一个顶点两次，（也就是说，最多通过|V|-1条边），while（1）的循环最多执行|V|-1次，因此，复杂度为O（|V|*|E|）。反之，如果存在从s可达的负圈，那么在第|V|次循环时也会更新d的值，因此可以检测负圈。如果一开始对所有的顶点i，都把d[i]初始化为0,那么可以查找出所有的负圈.
<pre class="lang:c++ decode:true">bool find_negative_loop(){
    memset(D,0,sizeof(D));
    for (int i = 0; i &lt; V; i += 1)
    {
    	for (int j = 0; j &lt; E; j += 1)
    	{
    		edge e=es[j];
    		if(D[e.to]&gt;D[e.from]+e.cost){
    			D[e.to]=D[e.from]+e.cost;
    			//如果第V次依旧更新了,则存在负圈
    			if(i==V-1)return true;
    		}
    	}
    }
    return false;
}</pre>
&nbsp;

### 2.Dijkstra算法

适用条件&amp;范围：
a) 单源最短路径(从源点s到其它所有顶点v);
b) 有向图&amp;无向图(无向图可以看作(u,v),(v,u)同属于边集E的有向图)
c) 所有边权非负(任取(i,j)∈E都有Wij≥0);
<pre class="minimize:true lang:c++ decode:true">struct edge
{
	int to,cost;
};
typedef pair&lt;int,int&gt; P;   //first是最短距离,second是顶点的编号.
int V;
const int MAX_V=100000,INF=INT32_MAX;
vector&lt;edge&gt; G[MAX_V];
int d[MAX_V];
void dijkstra(int s){
        //通过指定greater参数,堆按照first从小到达排列
	priority_queue&lt; P,vector&lt;P&gt;,greater&lt;P&gt; &gt; que;
	fill(d,d+V,INF);
	d[s]=0;
	que.push(P(0,s));
	while (!que.empty())
	{
		P p=que.top();que.pop();
		int v=p.second;
		if(p.first&gt;d[v])continue;
		for (unsigned int i = 0; i &lt; G[v].size(); i += 1)
		{
			edge e=G[v][i];
			if (d[e.to]&gt;d[v]+e.cost)
			{
				d[e.to]=d[v]+e.cost;
				que.push(P(d[e.to],e.to));
			}
		}
	}
}</pre>

## 任意两点间的最短路问题(Floyd-Warshall算法)

适用范围：
a) APSP(All Pairs Shortest Paths)
b) 稠密图效果最佳
c) 边权可正可负

利用DP来解决.对于任意两点,只使用顶点0~k和i,j的情况下,记i到j的最短路长度为d[k][i][j].k=-1时,认为只使用i和j,所以d[-1][i][j]=cost[i][j].

接下来把只使用顶点0~k的问题归结到只使用0~k-1:

只使用0~k-1时,分i到j的最短路正好经过顶点k一次和完全不经过顶点k的情况来讨论.不经过时,d[k][i][j]=d[k-1][i][j];经过时,d[k][i][j]=d[k-1][i][k]+d[k-1][k][i];
所以d[k][i][j]=min(d[k-1][i][j],d[k-1][i][k]+d[k-1][k][j]),显然可以用一个二维数组,不断进行d[i][j]=min(d[i][j],d[i][k]+d[k][j])更新来实现.
<pre class="lang:c++ decode:true">int d[MAX_V][MAX_V];  //d[u][v]表示边e=(u,v)的权值,不存在时设为INF,不过d[i][i]=0;
int V;  //顶点数;
void warshall_floyd(){
	for (int k = 0; k &lt; V; k += 1)
	{
		for (int i = 0; i &lt; V; i += 1)
		{
			for (int j = 0; j &lt; V; j += 1)
			{
				d[i][j]=min(d[i][j],d[i][k]+d[k][j]);
			}
		}
	}
	return;
}</pre>

## 路径还原

1.

输出时可能会遇到一点难处，我们记的是每个点“前面的”点是什么，输出却要从最前面往最后面输，这不好办。其实很好办，见如下递归方法
<div class="UBBPanel codePanel">
<div class="UBBContent">
<pre class="lang:c++ decode:true ">void PrintPath(int k){
    if( Path[k] ) PrintPath(Path[k]);
    fout&lt;&lt;k&lt;&lt;' ';
}</pre>
</div>
</div>
2.利用pre数组并且翻转进行最短路的还原.下面将dijkstra算法修改:
<pre class="minimize:true lang:c++ mark:17,23,36,43-49 range:17-49 decode:true">#include&lt;iostream&gt;
#include&lt;memory.h&gt;
#include&lt;stdio.h&gt;
#include&lt;queue&gt;
#include &lt;vector&gt;
#include&lt;algorithm&gt;

using namespace std;
struct edge
{
	int to,cost;
};
typedef pair&lt;int,int&gt; P;   //first是最短距离,second是顶点的编号.
int V;
const int MAX_V=100000,INF=INT32_MAX;
vector&lt;edge&gt; G[MAX_V];
int pre[MAX_V];          //
int d[MAX_V];

void dijkstra(int s){
	priority_queue&lt; P,vector&lt;P&gt;,greater&lt;P&gt; &gt; que;
	fill(d,d+V,INF);
	fill(pre,pre+V,-1);   //
	d[s]=0;
	que.push(P(0,s));
	while (!que.empty())
	{
		P p=que.top();que.pop();
		int v=p.second;
		if(p.first&gt;d[v])continue;
		for (unsigned int i = 0; i &lt; G[v].size(); i += 1)
		{
			edge e=G[v][i];
			if (d[e.to]&gt;d[v]+e.cost)
			{
				pre[e.to]=v;    //
				d[e.to]=d[v]+e.cost;
				que.push(P(d[e.to],e.to));
			}
		}
	}
}
vector&lt;int&gt; get_path(int t){     //
	vector&lt;int&gt; path;
	for (;t!=-1;t=pre[t])path.push_back(t);
	//翻转	
	reverse(path.begin(),path.end());    //#include&lt;algorithm&gt;
	return path;
}
int main()
{

	return 0;
}</pre>

## 最小生成树

### Prim算法(Dijksta的推广)

适用范围：
a) MST(Minimum Spanning Tree,最小生成树)
b) 无向图(有向图的是最小树形图)
c) 多用于稠密图
<pre class="minimize:true lang:c++ decode:true">#include&lt;iostream&gt;
#include&lt;memory.h&gt;
#include&lt;stdio.h&gt;
#include&lt;queue&gt;
#include &lt;vector&gt;
using namespace std;
int V,E;
const int MAX_V=10000,INF=INT32_MAX;    
int cost[MAX_V][MAX_V];   //表示边(u,v)的权值,如果不存在,初始化为INF
int mincost[MAX_V];
bool used[MAX_V];
int prim(){

	fill(mincost,mincost+V,INF);
	fill(used,used+V,false);
	mincost[0]=0;
	int ans=0;

	while(1){
		int v=-1;
		for (int i = 0; i &lt; V; i += 1)
		{
			if(!used[i]&amp;&amp;(v==-1||mincost[v]&gt;mincost[i]))v=i;
		}
		if(v==-1)break;
		used[v]=true;
		ans+=mincost[v];

		for (int i = 0; i &lt; V; i += 1)
		{
			mincost[i]=min(mincost[i],cost[v][i]);
		}
	}
	return ans;
}
int main()
{
	scanf("%d %d",&amp;V,&amp;E);
	int a,b,c;
	for (int i = 0; i &lt; V; i += 1)
	{
		for (int j = 0; j &lt; V; j += 1)
		{
			cost[i][j]=INF;
		}
	}
	for (int i = 0; i &lt; E; i += 1)
	{
		scanf("%d %d %d",&amp;a,&amp;b,&amp;c);
		cost[a][b]=c;
		cost[b][a]=c;
	}
	cout&lt;&lt;prim()&lt;&lt;endl;
	return 0;
}</pre>
这是朴素的prim算法,复杂度为O(V*V),可用堆来维护mincost数组优化到O(E*logV)但是过于繁杂,不推荐使用.

### Kruskal算法

适用范围：
a) MST(Minimum Spanning Tree,最小生成树)
b) 无向图(有向图的是最小树形图)
c) 多用于稀疏图
d) 边已经按权值排好序给出
<pre class="minimize:true lang:c++ decode:true ">#include&lt;iostream&gt;
#include&lt;memory.h&gt;
#include&lt;stdio.h&gt;
#include&lt;algorithm&gt;
using namespace std;
int V,E;
const int MAX_V=1000,MAX_E=1000,INF=INT32_MAX;
int par[MAX_V],ranks[MAX_V]; //并查集
int find(int x){
	if(par[x]==x)return x;
	else return par[x]=find(par[x]);
}
void unite(int x,int y){
	x=find(x);
	y=find(y);
	if(x==y)return;
	if(ranks[x]&lt;ranks[y]){
		par[x]=y;
	}else{
		par[y]=x;
		if(ranks[x]==ranks[y])ranks[x]++;
	}
}
bool same(int x,int y){
	return find(x)==find(y);
}
void init(int n){
	for (int i = 0; i &lt; n; i += 1)
	{
		par[i]=i;
		ranks[i]=1;
	}
}
struct edge   //
{
	int u,v,cost;
};
edge es[MAX_E];
bool cmp(edge a,edge b){
	return a.cost&lt;b.cost;
}

int kruskal(){
	sort(es,es+E,cmp);
	init(V);   //并查集初始化
	int res=0;
	for (int i = 0; i &lt; E; i += 1)
	{
		edge e=es[i];
		if(!same(e.u,e.v)){
			unite(e.u,e.v);
			res+=e.cost;
		}
	}
	return res;
}
int main()
{
	scanf("%d %d",&amp;V,&amp;E);
	int a,b,c;
	for (int i = 0; i &lt; E; i += 1)
	{
		scanf("%d %d %d",&amp;a,&amp;b,&amp;c);
		es[i].u=a;
		es[i].v=b;
		es[i].cost=c;
	}
	cout&lt;&lt;kruskal()&lt;&lt;endl;
	return 0;
}</pre>
Kruskal算法在排序上最费时,算法的复杂度为O(E*logV),理解起来很容易,注意写好并查集.

&nbsp;

参考博客:  [[ACM算法]图的基本知识](http://www.cnblogs.com/10jschen/archive/2012/08/15/2639650.html)

暂时这么多,剩下的以后再补~

#  补充:

## SPFA算法

求单源最短路的SPFA算法的全称是：Shortest Path Faster Algorithm，是西南交通大学段凡丁于1994年发表的。从名字我们就可以看出，这种算法在效率上一定有过人之处。很多时候，给定的图存在负权边，这时类似Dijkstra算法等便没有了用武之地，而Bellman-Ford算法的复杂度又过高，SPFA算法便派上用场了。简洁起见，我们约定加权有向图G不存在负权回路，即最短路径一定存在。如果某个点进入队列的次数超过N次则存在负环（SPFA无法处理带负环的图）。当然，我们可以在执行该算法前做一次拓扑排序，以判断是否存在负权回路，但这不是我们讨论的重点。我们用数组d记录每个结点的最短路径估计值，而且用邻接表来存储图G。我们采取的方法是动态逼近法：设立一个先进先出的队列用来保存待优化的结点，优化时每次取出队首结点u，并且用u点当前的最短路径估计值对离开u点所指向的结点v进行松弛操作，如果v点的最短路径估计值有所调整，且v点不在当前的队列中，就将v点放入队尾。这样不断从队列中取出结点来进行松弛操作，直至队列空为止。
<div class="para">定理：只要最短路径存在，上述SPFA算法必定能求出最小值。证明：每次 将点放入队尾，都是经过松弛操作达到的。换言之，每次的优化将会有某个点v的最短路径估计值d[v]变小。所以算法的执行会使d越来越小。由于我们假定图 中不存在负权回路，所以每个结点都有最短路径值。因此，算法不会无限执行下去，随着d值的逐渐变小，直到到达最短路径值时，算法结束，这时的最短路径估计 值就是对应结点的最短路径值。</div>
<div class="para"></div>
<div class="para">期望时间复杂度：O(me)， 其中m为所有顶点进队的平均次数，可以证明m一般小于等于2：“算法编程后实际运算情况表明m一般没有超过2n.事实上顶点入队次数m是一个不容易事先分 析出来的数,但它确是一个随图的不同而略有不同的常数.所谓常数,就是与e无关,与n也无关,仅与边的权值分布有关.一旦图确定,权值确定,原点确定,m 就是一个确定的常数.所以SPFA算法复杂度为O(e).证毕."（SPFA的论文）不过，这个证明是非常不严谨甚至错误的，事实上在bellman算法的论文中已有这方面的内容，所以国际上一般不承认SPFA算法。</div>
<div class="para"></div>
<div class="para">对SPFA的一个很直观的理解就是由无权图的BFS转 化而来。在无权图中，BFS首先到达的顶点所经历的路径一定是最短路(也就是经过的最少顶点数)，所以此时利用数组记录节点访问可以使每个顶点只进队一 次，但在带权图中，最先到达的顶点所计算出来的路径不一定是最短路。一个解决方法是放弃数组，此时所需时间自然就是指数级的，所以我们不能放弃数组，而是 在处理一个已经在队列中且当前所得的路径比原来更好的顶点时，直接更新最优解。</div>
<div class="para"></div>
<div class="para">SPFA算法有两个优化策略SLF和LLL——SLF：Small Label First 策略，设要加入的节点是j，队首元素为i，若dist(j)&lt;dist(i)，则将j插入队首，否则插入队尾； LLL：Large Label Last 策略，设队首元素为i，队列中所有dist值的平均值为x，若dist(i)&gt;x则将i插入到队尾，查找下一元素，直到找到某一i使得 dist(i)&lt;=x，则将i出队进行松弛操作。 SLF 可使速度提高 15 ~ 20%；SLF + LLL 可提高约 50%。 在实际的应用中SPFA的算法时间效率不是很稳定，为了避免最坏情况的出现，通常使用效率更加稳定的Dijkstra算法。</div>
<div class="para"></div>
<div class="para">**SPFA的两种写法，bfs和dfs，bfs判别负环不稳定，相当于限深度搜索，但是设置得好的话还是没问题的，dfs的话判断负环很快**</div>
<div class="para"></div>
<pre class="lang:c++ decode:true">int spfa_bfs(int s)
{
    queue &lt;int&gt; q;
    memset(d,0x3f,sizeof(d));
    d[s]=0;
    memset(c,0,sizeof(c));
    memset(vis,0,sizeof(vis));

    q.push(s);  vis[s]=1; c[s]=1;
    //顶点入队vis要做标记，另外要统计顶点的入队次数
    int OK=1;
    while(!q.empty())
    {
        int x;
        x=q.front(); q.pop();  vis[x]=0;
        //队头元素出队，并且消除标记
        for(int k=f[x]; k!=0; k=nnext[k]) //遍历顶点x的邻接表
        {
            int y=v[k];
            if( d[x]+w[k] &lt; d[y])
            {
                d[y]=d[x]+w[k];  //松弛
                if(!vis[y])  //顶点y不在队内
                {
                    vis[y]=1;    //标记
                    c[y]++;      //统计次数
                    q.push(y);   //入队
                    if(c[y]&gt;NN)  //超过入队次数上限，说明有负环
                        return OK=0;
                }
            }
        }
    }

    return OK;

}</pre>
<pre class="lang:c++ decode:true ">int spfa_dfs(int u)
{
    vis[u]=1;
    for(int k=f[u]; k!=0; k=e[k].next)
    {
        int v=e[k].v,w=e[k].w;
        if( d[u]+w &lt; d[v] )
        {
            d[v]=d[u]+w;
            if(!vis[v])
            {
                if(spfa_dfs(v))
                    return 1;
            }
            else
                return 1;
        }
    }
    vis[u]=0;
    return 0;
}</pre>
&nbsp;

&nbsp;

&nbsp;