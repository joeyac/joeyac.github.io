---
title: bitset的使用(转)
tags:
  - bitset
  - 总结
id: 224
categories:
  - ACM
date: 2016-03-09 16:55:17
---

[原文链接](http://blog.163.com/lixiangqiu_9202/blog/static/53575037201251121331412/)

有些程序要处理二进制位的有序集，每个位可能包含的是0（关）或1（开）的值。位是用来保存一组项或条件的<span lang="EN-US">yes</span>/<span lang="EN-US">no</span>信息（有时也称标志）的简洁方法。标准库提供了<span lang="EN-US">bitset</span>类使得处理位集合更容易一些。要使用<span lang="EN-US">bitset</span>类就必须要包含相关的头文件。在本书提供的例子中，假设都使用了<span lang="EN-US">std::bitset</span>的<span lang="EN-US">using</span>声明：

<span lang="EN-US">＃i nclude &lt;bitset&gt;</span>

<span lang="EN-US">using std::bitset;</span>

3.5.1  **bitset**的定义和初始化

表<span lang="EN-US">3-6</span>列出了<span lang="EN-US">bitset</span>的构造函数。类似于<span lang="EN-US">vector</span>，<span lang="EN-US">bitset</span>类是一种类模板；而与<span lang="EN-US">vector</span>不一样的是<span lang="EN-US">bitset</span>类型对象的区别仅在其长度而不在其类型。在定义<span lang="EN-US">bitset</span>时，要明确<span lang="EN-US">bitset</span>含有多少位，须在尖括号内给出它的长度值：

<span lang="EN-US">bitset</span>&lt;<span lang="EN-US">32</span>&gt; <span lang="EN-US">bitvec</span>; //<span lang="EN-US">32</span>位，全为<span lang="EN-US">0</span>。

给出的长度值必须是常量表达式（2.7节）。正如这里给出的，长度值必须定义为整型字面值常量或是已用常量值初始化的整数类型的<span lang="EN-US">const</span>对象。

这条语句把<span lang="EN-US">bitvec</span>定义为含有<span lang="EN-US">32</span>个位的<span lang="EN-US">bitset</span>对象。和<span lang="EN-US">vector</span>的元素一样，<span lang="EN-US">bitset</span>中的位是没有命名的，程序员只能按位置来访问它们。位集合的位置编号从0开始，因此，<span lang="EN-US">bitvec</span>的位序是从0到31。以0位开始的位串是低阶位（<span lang="EN-US">low-order bit</span>），以31位结束的位串是高阶位(<span lang="EN-US">high-order bit</span>)。

表<span lang="EN-US">3-6  </span>初始化<span lang="EN-US">bitset</span>对象的方法
<div>
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="170"><span lang="EN-US">bitset&lt;n&gt; b;</span></td>
<td width="382"><span lang="EN-US">b</span>有<span lang="EN-US">n</span>位，每位都为<span lang="EN-US">0</span></td>
</tr>
<tr>
<td width="170"><span lang="EN-US">bitset</span><span lang="EN-US">&lt;</span><span lang="EN-US">n</span><span lang="EN-US">&gt; </span><span lang="EN-US">b</span><span lang="EN-US">(</span><span lang="EN-US">u</span><span lang="EN-US">);</span></td>
<td width="382"><span lang="EN-US">b</span>是<span lang="EN-US">unsigned long</span>型<span lang="EN-US">u</span>的一个副本</td>
</tr>
<tr>
<td width="170"><span lang="EN-US">bitset</span><span lang="EN-US">&lt;</span><span lang="EN-US">n</span><span lang="EN-US">&gt; </span><span lang="EN-US">b</span><span lang="EN-US">(</span><span lang="EN-US">s</span><span lang="EN-US">);</span></td>
<td width="382"><span lang="EN-US">b</span>是<span lang="EN-US">string</span>对象<span lang="EN-US">s</span>中含有的位串的副本</td>
</tr>
<tr>
<td width="170"><span lang="EN-US">bitset</span><span lang="EN-US">&lt;</span><span lang="EN-US">n</span><span lang="EN-US">&gt; </span><span lang="EN-US">b</span><span lang="EN-US">(</span><span lang="EN-US">s</span><span lang="EN-US">, </span><span lang="EN-US">pos</span><span lang="EN-US">, </span><span lang="EN-US">n</span><span lang="EN-US">);</span></td>
<td width="382"><span lang="EN-US">b</span>是<span lang="EN-US">s</span>中从位置<span lang="EN-US">pos</span>开始的<span lang="EN-US">n</span>个位的副本</td>
</tr>
</tbody>
</table>
</div>
<span lang="EN-US">1. </span>用**<span lang="EN-US">unsigned</span>**值初始化**<span lang="EN-US">bitset</span>**对象

当用<span lang="EN-US">unsigned long</span>值作为<span lang="EN-US">bitset</span>对象的初始值时，该值将转化为二进制的位模式。而<span lang="EN-US">bitset</span>对象中的位集作为这种位模式的副本。如果<span lang="EN-US">bitset</span>类型长度大于<span lang="EN-US">unsigned long</span>值的二进制位数，则其余的高阶位置为<span lang="EN-US">0</span>；如果<span lang="EN-US">bitet</span>类型长度小于<span lang="EN-US">unsigned long</span>值的二进制位数，则只使用<span lang="EN-US">unsigned</span>值中的低阶位，超过<span lang="EN-US">bitet</span>类型长度的高阶位将被丢弃。

在<span lang="EN-US">32</span>位<span lang="EN-US">unsigned long</span>的机器上，十六进制值<span lang="EN-US">0xffff</span>表示为二进制位就是十六个<span lang="EN-US">1</span>和十六个<span lang="EN-US">0</span>（每个<span lang="EN-US">0xf</span>可表示为<span lang="EN-US">1111</span>）。可以用<span lang="EN-US">0xffff</span>初始化<span lang="EN-US">bitset</span>对象：

<span lang="EN-US">// _bitvec1__ _</span><span lang="EN-US">is smaller than the initializer</span>

<span lang="EN-US">bitset&lt;16&gt; bitvec1(0xffff);          // </span><span lang="EN-US">bits 0 ... 15 are set to 1</span>

<span lang="EN-US">// _bitvec2__ _</span><span lang="EN-US">same size as initializer</span>

<span lang="EN-US">bitset&lt;32&gt; bitvec2(0xffff);          // </span><span lang="EN-US">bits 0 ... 15 are set to 1; 16 ... 31 are 0</span>

<span lang="EN-US">// </span><span lang="EN-US">on a 32-bit machine, bits 0 to 31 initialized from</span>_<span lang="EN-US"> 0xffff</span>_

<span lang="EN-US">bitset&lt;128&gt; bitvec3(0xffff);         // </span><span lang="EN-US">bits 32 through 127 initialized to zero</span>

上面的三个例子中，<span lang="EN-US">0</span>到<span lang="EN-US">15</span>位都置为<span lang="EN-US">1</span>。由于<span lang="EN-US">bitvec1</span>位数少于<span lang="EN-US">unsigned long</span>的位数，因此<span lang="EN-US">bitvec1</span>的初始值的高阶位被丢弃。<span lang="EN-US">bitvec2</span>和<span lang="EN-US">unsigned long</span>长度相同，因此所有位正好放置了初始值。<span lang="EN-US">bitvec3</span>长度大于<span lang="EN-US">32</span>，<span lang="EN-US">31</span>位以上的高阶位就被置为<span lang="EN-US">0</span>。

<span lang="EN-US">2. </span>用**<span lang="EN-US">string</span>**对象初始化**<span lang="EN-US">bitset</span>**对象

当用<span lang="EN-US">string</span>对象初始化<span lang="EN-US">bitset</span>对象时，<span lang="EN-US">string</span>对象直接表示为位模式。从<span lang="EN-US">string</span>对象读入位集的顺序是从右向左：

<span lang="EN-US">string strval("1100");</span>

<span lang="EN-US">bitset&lt;32&gt; bitvec4(strval);</span>

<span lang="EN-US">bitvec4</span>的位模式中第<span lang="EN-US">2</span>和<span lang="EN-US">3</span>的位置为<span lang="EN-US">1</span>，其余位置都为<span lang="EN-US">0</span>。如果<span lang="EN-US">string</span>对象的字符个数小于<span lang="EN-US">bitset</span>类型的长度，则高阶位将置为<span lang="EN-US">0</span>。

<span lang="EN-US">string</span>对象和<span lang="EN-US">bitset</span>对象之间是反向转化的：<span lang="EN-US">string</span>对象的最右边字符（即下标最大的那个字符）用来初始化<span lang="EN-US">bitset</span>对象的低阶位（即下标为<span lang="EN-US">0</span>的位）。当用<span lang="EN-US">string</span>对象初始化<span lang="EN-US">bitset</span>对象时，记住这一差别很重要。

不一定要把整个<span lang="EN-US">string</span>对象都作为<span lang="EN-US">bitset</span>对象的初始值。相反，可以只用某个子串作为初始值：

<span lang="EN-US">string str("1111111000000011001101");</span>

<span lang="EN-US">bitset&lt;32&gt; bitvec5(str, 5, 4); // </span><span lang="EN-US">4 bits starting at</span>_<span lang="EN-US"> str[5], 1100</span>_

<span lang="EN-US">bitset&lt;32&gt; bitvec6(str, str.size() - 4);     // </span><span lang="EN-US">use last 4 characters</span>

这里用<span lang="EN-US">str</span>中从<span lang="EN-US">str[5]</span>开始包含四个字符的子串来初始化<span lang="EN-US">bitvec5</span>。照常，初始化<span lang="EN-US">bitset</span>对象时总是从子串最右边结尾字符开始的，<span lang="EN-US">bitvec5</span>的从<span lang="EN-US">0</span>到<span lang="EN-US">3</span>的二进制位置为<span lang="EN-US">1100</span>，其他二进制位都置为<span lang="EN-US">0</span>。如果省略第三个参数则意味着取从开始位置一直到<span lang="EN-US">string</span>末尾的所有字符。本例中，取出<span lang="EN-US">str</span>末尾的四位来对<span lang="EN-US">bitvec6</span>的低四位进行初始化。<span lang="EN-US">bitvec6</span>其余的位初始化为<span lang="EN-US">0</span>。这些初始化过程的图示如下：

多种<span lang="EN-US">bitset</span>操作（表<span lang="EN-US">3-7</span>）用来测试或设置<span lang="EN-US">bitset</span>对象中的单个或多个二进制位：

表<span lang="EN-US">3-7  </span>**<span lang="EN-US">bitset</span>**操作
<div>
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="138"><span lang="EN-US">b.any()</span></td>
<td width="414"><span lang="EN-US">b</span>中是否存在置为<span lang="EN-US">1</span>的二进制位？</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">none</span><span lang="EN-US">()</span></td>
<td width="414"><span lang="EN-US">b</span>中不存在置为<span lang="EN-US">1</span>的二进制位吗？</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">count</span><span lang="EN-US">()</span></td>
<td width="414"><span lang="EN-US">b</span>中置为<span lang="EN-US">1</span>的二进制位的个数</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">size</span><span lang="EN-US">()</span></td>
<td width="414"><span lang="EN-US">b</span>中二进制位的个数</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">[</span><span lang="EN-US">pos</span><span lang="EN-US">]</span></td>
<td width="414">访问<span lang="EN-US">b</span>中在<span lang="EN-US">pos</span>处的二进制位</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">test</span><span lang="EN-US">(</span><span lang="EN-US">pos</span><span lang="EN-US">)</span></td>
<td width="414"><span lang="EN-US">b</span>中在<span lang="EN-US">pos</span>处的二进制位是否为<span lang="EN-US">1</span>？</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">set</span><span lang="EN-US">()</span></td>
<td width="414">把<span lang="EN-US">b</span>中所有二进制位都置为<span lang="EN-US">1</span></td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">set</span><span lang="EN-US">(</span><span lang="EN-US">pos</span><span lang="EN-US">)</span></td>
<td width="414">把<span lang="EN-US">b</span>中在<span lang="EN-US">pos</span>处的二进制位置为<span lang="EN-US">1</span></td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">reset</span><span lang="EN-US">()</span></td>
<td width="414">把<span lang="EN-US">b</span>中所有二进制位都置为<span lang="EN-US">0</span></td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">reset</span><span lang="EN-US">(</span><span lang="EN-US">pos</span><span lang="EN-US">)</span></td>
<td width="414">把<span lang="EN-US">b</span>中在<span lang="EN-US">pos</span>处的二进制位置为<span lang="EN-US">0</span></td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">flip</span><span lang="EN-US">()</span></td>
<td width="414">把<span lang="EN-US">b</span>中所有二进制位逐位取反</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">flip</span><span lang="EN-US">(</span><span lang="EN-US">pos</span><span lang="EN-US">)</span></td>
<td width="414">把<span lang="EN-US">b</span>中在<span lang="EN-US">pos</span>处的二进制位取反</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">b</span><span lang="EN-US">.</span><span lang="EN-US">to</span><span lang="EN-US">_</span><span lang="EN-US">ulong</span><span lang="EN-US">()</span></td>
<td width="414">用<span lang="EN-US">b</span>中同样的二进制位返回一个<span lang="EN-US">unsigned long</span>值</td>
</tr>
<tr>
<td width="138"><span lang="EN-US">os</span><span lang="EN-US"> &lt;&lt; </span><span lang="EN-US">b</span></td>
<td width="414">把<span lang="EN-US">b</span>中的位集输出到<span lang="EN-US">os</span>流</td>
</tr>
</tbody>
</table>
</div>
<span lang="EN-US">1. </span>测试整个**<span lang="EN-US">bitset</span>**对象

如果<span lang="EN-US">bitset</span>对象中有一个或多个二进制位置为<span lang="EN-US">1</span>，则<span lang="EN-US">any</span>操作返回<span lang="EN-US">true</span>，也就是说，其返回值等于<span lang="EN-US">1</span><span lang="EN-US">;</span>相反，如果<span lang="EN-US">bitset</span>对象中的二进制位全为<span lang="EN-US">0</span><span lang="EN-US">,</span>则<span lang="EN-US">none</span>操作返回<span lang="EN-US">true</span>。

<span lang="EN-US">bitset&lt;32&gt; bitvec; // </span><span lang="EN-US">32 bits, all zero</span>

<span lang="EN-US">bool is_set = bitvec.any();            // </span><span lang="EN-US">false, all bits are zero</span>

<span lang="EN-US">bool is_not_set = bitvec.none();      // </span><span lang="EN-US">true, all bits are zero</span>

如果需要知道置为<span lang="EN-US">1</span>的二进制位的个数，可以使用<span lang="EN-US">count</span>操作，该操作返回置为<span lang="EN-US">1</span>的二进制位的个数：

<span lang="EN-US">size_t bits_set = bitvec.count(); // </span><span lang="EN-US">returns number of bits that are on</span>

<span lang="EN-US">count</span>操作的返回类型是标准库中命名为**<span lang="EN-US">size_t</span>**的类型。<span lang="EN-US">size_t</span>类型定义在<span lang="EN-US">cstddef</span>头文件中，该文件是<span lang="EN-US">C</span>标准库的头文件<span lang="EN-US">stddef.h</span>的<span lang="EN-US">C</span><span lang="EN-US">++</span>版本。它是一个与机器相关的_<span lang="EN-US">unsigned</span>_类型，大小可以保证存储内存中对象。

与<span lang="EN-US">vector</span>和<span lang="EN-US">string</span>中的<span lang="EN-US">size</span>操作一样，<span lang="EN-US">bitset</span>的<span lang="EN-US">size</span>操作返回<span lang="EN-US">bitset</span>对象中二进制位的个数，返回值的类型是<span lang="EN-US">size</span><span lang="EN-US">_</span><span lang="EN-US">t</span><span lang="EN-US">:</span>

<span lang="EN-US">size_t sz = bitvec.size(); // </span><span lang="EN-US">returns </span>_<span lang="EN-US">32</span>_

<span lang="EN-US">2. </span>访问**<span lang="EN-US">bitset</span>**对象中的位

可以用下标操作符来读或写某个索引位置的二进制位，同样地，也可以用下标操作符测试给定二进制位的值或设置某个二进制位的值：

<span lang="EN-US">// </span><span lang="EN-US">assign 1 to even numbered bits</span>

<span lang="EN-US">for (int index = 0; index != 32; index += 2)</span>

<span lang="EN-US">           bitvec[index] = 1;</span>

上面的循环把<span lang="EN-US">bitvec</span>中的偶数下标的位都置为<span lang="EN-US">1</span>。

除了用下标操作符，还可以用<span lang="EN-US">set</span>、<span lang="EN-US">test</span>和<span lang="EN-US">reset</span>操作来测试或设置给定二进制位的值：

<span lang="EN-US">// </span><span lang="EN-US">equivalent loop using set operation</span>

<span lang="EN-US">for (int index = 0; index != 32; index += 2)</span>

<span lang="EN-US">           bitvec.set(index);</span>

为了测试某个二进制位是否为<span lang="EN-US">1</span>，可以用<span lang="EN-US">test</span>操作或者测试下标操作符的返回值：

<span lang="EN-US">if (bitvec.test(i))</span>

<span lang="EN-US">    // </span><span lang="EN-US">bitvec[i] is on</span>

<span lang="EN-US">// </span><span lang="EN-US">equivalent test using subscript</span>

<span lang="EN-US">if (bitvec[i])</span>

<span lang="EN-US">    // </span><span lang="EN-US">bitvec[i] is on</span>

如果下标操作符测试的二进制位为<span lang="EN-US">1</span>，则返回的测试值的结果为<span lang="EN-US">true</span>，否则返回<span lang="EN-US">false</span>。

<span lang="EN-US">3. </span>对整个**<span lang="EN-US">bitset</span>**对象进行设置

<span lang="EN-US">set</span>和<span lang="EN-US">reset</span>操作分别用来对整个<span lang="EN-US">bitset</span>对象的所有二进制位全置<span lang="EN-US">1</span>和全置<span lang="EN-US">0</span>：

<span lang="EN-US">bitvec.reset();    // </span><span lang="EN-US">set all the bits to 0.</span>

<span lang="EN-US">bitvec.set();      // </span><span lang="EN-US">set all the bits to 1</span>

<span lang="EN-US">flip</span>操作可以对<span lang="EN-US">bitset</span>对象的所有位或个别位按位取反：

<span lang="EN-US">bitvec.flip(0);   // </span><span lang="EN-US">reverses value of first bit</span>

<span lang="EN-US">bitvec[0].flip(); // </span><span lang="EN-US">also reverses the first bit</span>

<span lang="EN-US">bitvec.flip();    // </span><span lang="EN-US">reverses value of all bits</span>

<span lang="EN-US">4. </span>获取<span lang="EN-US">bitset</span>对象的值

<span lang="EN-US">to_ulong</span>操作返回一个<span lang="EN-US">unsigned</span><span lang="EN-US"> </span><span lang="EN-US">long</span>值，该值与<span lang="EN-US">bitset</span>对象的位模式存储值相同。仅当<span lang="EN-US">bitset</span>类型的长度小于或等于<span lang="EN-US">unsigned</span><span lang="EN-US"> </span><span lang="EN-US">long</span>的长度时，才可以使用<span lang="EN-US">to_ulong</span>操作：

<span lang="EN-US">unsigned long ulong = bitvec3.to_ulong();</span>

<span lang="EN-US">cout &lt;&lt; "ulong = " &lt;&lt; ulong &lt;&lt; endl;</span>

<span lang="EN-US">to_ulong</span>操作主要用于把<span lang="EN-US">bitset</span>对象转到<span lang="EN-US">C</span>风格或标准<span lang="EN-US">C</span><span lang="EN-US">++</span>之前风格的程序上。如果<span lang="EN-US">bitset</span>对象包含的二进制位数超过<span lang="EN-US">unsigned long</span>的长度，将会产生运行时异常。本书将在<span lang="EN-US">6.13</span>节介绍异常（<span lang="EN-US">exception</span>），并在<span lang="EN-US">17.1</span>节中详细地讨论它。

<span lang="EN-US">5. </span>输出二进制位

可以用输出操作符输出<span lang="EN-US">bitset</span>对象中的位模式：

<span lang="EN-US">bitset&lt;32&gt; bitvec2(0xffff); // </span><span lang="EN-US">bits 0 ... 15 are set to 1; 16 ... 31 are 0</span>

<span lang="EN-US">cout &lt;&lt; "bitvec2: " &lt;&lt; bitvec2 &lt;&lt; endl;</span>

输出结果为：

**<span lang="EN-US">bitvec2: 00000000000000001111111111111111</span>**

<span lang="EN-US">6. </span>使用位操作符

<span lang="EN-US">bitset</span>类也支持内置的位操作符。<span lang="EN-US">C++</span>定义的这些操作符都只适用于整型操作数，它们所提供的操作类似于本节所介绍的<span lang="EN-US">bitset</span>操作。

&nbsp;