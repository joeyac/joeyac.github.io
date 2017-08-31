---
title: Find the Duplicate Number
id: 424
categories:
  - ACM
date: 2016-08-31 14:18:56
tags:
---

最近偶然发现了一个oj（[Leetcode](https://leetcode.com/)）\(^o^)/上面的题目很有意思，各种脑洞hhhh刷上瘾了，一些比较好的题目决定做记录在博客中～
> [287\. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)
> 
> 
> Given an array _nums_ containing _n_ + 1 integers where each integer is between 1 and _n_ (inclusive), prove that at least one duplicate number must exist. Assume that there is only one duplicate number, find the duplicate one.
> 
> 
> **Note:**
> 
> 
> 1.  You **must not** modify the array (assume the array is read only).
> 2.  You must use only constant, _O_(1) extra space.
> 3.  Your runtime complexity should be less than `O(n<sup>2</sup>)`.
> 4.  There is only one duplicate number in the array, but it could be repeated more than once.
首先是要证明至少有一个数会重复，这个由鸽巢原理很容易得出。其次题目保证只有一个数字会重复，但是可能会重复多次。

给出的四个限制把常规的四种解法都禁掉了，苦思一下午都没想出什么好解法，然后在讨论版发现了三种解法（时间复杂度分别为O(32*n),O(nlogn),O(n)），一种比一种 =  =╮(╯▽╰)╭下面来观摩一下：

solution 1：

由于传入的是一个vector&lt;int&gt;，也就是说2进制表示每个数都不超过32位，我们来对二进制下的每一位来处理，例如现在来处理第p位，计算输入的动态数组中每个数中第p位的和（存储到变量b）以及数组[1, 2, 3, ..., n]中每个数第p位的和（存储到变量a），如果b比a大，那就说明那个重复的数在第p位上为1，否则b不可能比a大。通过这种方式得出重复数值的每一位。

code(时间复杂度O(32*n))：

<pre class="lang:c++ decode:true">	int findDuplicate1(vector&lt;int&gt;&amp; nums) {   //O(32*N)
		int n = nums.size()-1, res = 0;
		for (int p = 0; p &lt; 32; ++ p) {
		    int bit = (1 &lt;&lt; p), a = 0, b = 0;
		    for (int i = 0; i &lt;= n; ++ i) {
		        if (i &gt; 0 &amp;&amp; (i &amp; bit) &gt; 0) ++a;//为了便于理解，加上了i&gt;0的判断
                                                        //但其实有点多余，因为当i=0时i&amp;bit=0
		        if ((nums[i] &amp; bit) &gt; 0) ++b;
		    }
		    if (b &gt; a) res += bit;
		}
		return res;
    }</pre>
solution 2:

基于二分搜索以及鸽巢原理（抽屉原理）的方法。

一开始我们搜索的范围为1到n之间的数字，每次选择搜索范围中间的那个数mid，并且计算输入数组中，所有不大于mid的数的个数（记为count）。然后，如果count比mid大，那么我们就可以将搜索范围缩小到[1，mid]而不是 [mid+1，n]，由鸽巢原理（[https://en.wikipedia.org/wiki/Pigeonhole_principle](https://en.wikipedia.org/wiki/Pigeonhole_principle)），很容易可以得出，[1,mid]中一定存在某个数至少出现了两次，然后题目保证只有一个且必定有一个数重复，于是可以不断进行二分搜索得出答案，复杂度为O（nlogn）。n显然不大于2的32次方(4294967296)，然而很有趣的是log4294967296仅仅为9.633，也就是说这个算法的复杂度要比前者高那么一些。

Code(时间复杂度O(nlogn))：

<pre class="lang:c++ decode:true ">    int findDuplicate2(vector&lt;int&gt;&amp; nums) {  //O(nlogn)
        int len=nums.size();
        int left=0,right=len-1,mid;
        int count;
        while(left&lt;right){
            mid=(left+right)/2;
            count=0;
            for(int i=0;i&lt;len;i++){
                if(nums[i]&lt;=mid)count++;
            }
            if(count&gt;mid)right=mid;
            else left=mid+1;
        }
        return left;
    }</pre>
solution 3：

最后一个，重头戏来了～

首先附上原作者的解答链接以及表达我不能用语言来形容的敬佩：[http://keithschwarz.com/interesting/code/find-duplicate/FindDuplicate.python.html](http://keithschwarz.com/interesting/code/find-duplicate/FindDuplicate.python.html)

&nbsp;