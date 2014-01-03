---
author: TheC
comments: true
date: 2012-01-23 10:07:37+00:00
layout: post
slug: hashtablecollisionsattack
title: 一种新型的拒绝服务攻击方式
wordpress_id: 258
categories:
- ACFUN
---

最近针对服务器有一种新型的玩法即哈希表碰撞攻击。多数语言都存在这一漏洞，而且到目前为止许多语言并未修补这一漏洞，攻击者能以极小的带宽支出达到使大型web服务器宕机。
<!-- more -->
受影响的语言：
PHP(在5.3.9加了一个临时的max_input_vars变量，可以认为已修复)
ASP.NET(微软通过自动更新改变了.Net Framework的算法，修复了此漏洞。http://technet.microsoft.com/en-us/security/bulletin/ms11-100)
Python(未修复，有个别Framework尝试修复，但仍可绕过)
Rubinius(未修复)
Ruby(1.8.7-p356已修复)
Apache Geronimo(未修复)
Apache Tomcat (5.5.34,6.0.34,7.0.22均已修复，但是跟php一样只是单纯的限制了提交的变量数量。鄙视一下Oracle,因为Oracle拒绝像微软那样在Java本身上做出升级)
Oracle Glassfish(已修复)
Jetty(未修复)
Plone(未修复)
Rack(未修复)
V8 JavaScript Engine(未修复)

下面来看看该攻击的原理。(这里主要以PHP和Lua为例。)
哈希表是一种查找效率极高的数据结构，很多语言都在内部实现了哈希表。PHP中的哈希表是一种极为重要的数据结构，不但用于表示Array数据类型，还在Zend虚拟机内部用于存储上下文环境信息（执行上下文的变量及函数均使用哈希表结构存储）。理想情况下哈希表插入和查找操作的时间复杂度均为O(1)，任何一个数据项可以在一个与哈希表长度无关的时间内计算出一个哈希值（key），然后在常量时间内定位到一个桶（术语bucket，表示哈希表中的一个位置）。当然这是理想情况下，因为任何哈希表的长度都是有限的，所以一定存在不同的数据项具有相同哈希值的情况，此时不同数据项被定为到同一个桶，称为碰撞（collision）。PHP数组为纯哈希表，无通常意义上的数组部分，所有下标均作为key看待，使用按哈希值取模分桶方式保存键值对，靠外部拉链解决冲突。PHP中，初始分桶数为2^3=8个（_zend_hash_init@zend_hash.c:140），插入哈希表的键值对数量超过分桶数后，分桶数量倍增并将所有键值对重新哈希（zend_hash_do_resize@zend_hash.c:418），数值（或数值字符串）下标的key被转换成整数后直接当作哈希值使用，即H(n)=n（注意这一点，是造成碰撞的关键），而字符串哈希算法为DJBX33A，全部键内容均参与哈希值计算（zend_inline_hash_func@zend_hash.h:261，这里也是比较重要的地方，和ASP.net不一样，.net Framework可以任意更改算法避免类似碰撞，对于PHP来说则是不可能的）。
综上所述，PHP是使用单链表存储碰撞的数据，因此实际上PHP哈希表的平均查找复杂度为O(L)，其中L为桶链表的平均长度；而最坏复杂度为O(N)，此时所有数据全部碰撞，哈希表退化成单链表。通过提交精心构造数据，人为将哈希表变成一个退化的单链表，而这样一来，将对服务器造成很大的性能开销，此时哈希表各种操作的时间均提升了一个数量级，从而达到拒绝服务攻击（DoS）的目的。
怎样构成所谓的“精心构造的字符串”呢？我们以PHP和Lua为例。

前面讲过，PHP中使用一个叫Backet的结构体表示桶，同一哈希值的所有桶被组织为一个单链表。哈希表使用HashTable结构体表示。相关源码在zend/Zend_hash.h下：
{% highlight cpp %}
typedef struct bucket {
    ulong h;                        /\* Used for numeric indexing \*/ //用于存储原始key
    uint nKeyLength;
    void \*pData;
    void \*pDataPtr;
    struct bucket \*pListNext;
    struct bucket \*pListLast;
    struct bucket \*pNext;
    struct bucket \*pLast;
    char arKey[1]; /\* Must be last element \*/
} Bucket;
 
typedef struct _hashtable {
    uint nTableSize;
    uint nTableMask;
    uint nNumOfElements;
    ulong nNextFreeElement;
    Bucket \*pInternalPointer;   /\* Used for element traversal \*/
    Bucket \*pListHead;
    Bucket \*pListTail;
    Bucket \*\*arBuckets; //指向一个指针数组，其中每个元素是一个指向Bucket链表的头指针
    dtor_func_t pDestructor;
    zend_bool persistent;
    unsigned char nApplyCount;
    zend_bool bApplyProtection;
#if ZEND_DEBUG
    int inconsistent;
#endif
} 
{% endhighlight %}
PHP哈希表最小容量是8（2^3），最大容量是0×80000000（2^31），并向2的整数次幂圆整（即长度会自动扩展为2的整数次幂，如13个元素的哈希表长度为16；100个元素的哈希表长度为128）。nTableMask被初始化为哈希表长度（圆整后）减1。具体代码在zend/Zend_hash.c的_zend_hash_init函数中,代码很长这里不再贴出。
在PHP Hash算法中，如果原始key不是字符串，简单将数据的原始key与HashTable的nTableMask进行按位与，否则，则首先使用Times33算法将字符串转为整形再与nTableMask按位与。
可以看看PHP怎么查找Hash表：
{% highlight cpp %}
ZEND_API int zend_hash_index_find(const HashTable \*ht, ulong h, void \*\*pData)
{
    uint nIndex;
    Bucket \*p;
 
    IS_CONSISTENT(ht);
 
    nIndex = h & ht->nTableMask;
 
    p = ht->arBuckets[nIndex];
    while (p != NULL) {
        if ((p->h == h) && (p->nKeyLength == 0)) {
            \*pData = p->pData;
            return SUCCESS;
        }
        p = p->pNext;
    }
    return FAILURE;
}
 
ZEND_API int zend_hash_find(const HashTable \*ht, const char \*arKey, uint nKeyLength, void \*\*pData)
{
    ulong h;
    uint nIndex;
    Bucket \*p;
 
    IS_CONSISTENT(ht);
 
    h = zend_inline_hash_func(arKey, nKeyLength);
    nIndex = h & ht->nTableMask;
 
    p = ht->arBuckets[nIndex];
    while (p != NULL) {
        if ((p->h == h) && (p->nKeyLength == nKeyLength)) {
            if (!memcmp(p->arKey, arKey, nKeyLength)) {
                \*pData = p->pData;
                return SUCCESS;
            }
        }
        p = p->pNext;
    }
    return FAILURE;
}
{% endhighlight %}
知道了PHP内部哈希表的算法，就可以利用其原理构造用于攻击的数据。一种最简单的方法是利用掩码规律制造碰撞。上文提到Zend HashTable的长度nTableSize会被圆整为2的整数次幂，假设我们构造一个2^16的哈希表，则nTableSize的二进制表示为：1 0000 0000 0000 0000，而nTableMask = nTableSize – 1为：0 1111 1111 1111 1111。接下来，可以以0为初始值，以2^16为步长，制造足够多的数据，可以得到如下推测：

0000 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0001 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0010 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0011 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0
0100 0000 0000 0000 0000 & 0 1111 1111 1111 1111 = 0

……

概况来说只要保证后16位均为0，则与掩码位于后得到的哈希值全部碰撞在位置0。

下面一段代码就是利用掩码规律实现HashTable攻击的代码，网上非常有名：
{% highlight php %}
<?php
 
$size = pow(2, 16);
 
$startTime = microtime(true);
 
$array = array();
for ($key = 0, $maxKey = ($size - 1) \* $size; $key <= $maxKey; $key += $size) {
    $array[$key] = 0;
}
 
$endTime = microtime(true);
 
echo $endTime - $startTime, ' seconds', "n";
{% endhighlight %}
与之对比的有一段代码：
{% highlight php %}
<?php
 
$size = pow(2, 16);
 
$startTime = microtime(true);
 
$array = array();
for ($key = 0, $maxKey = ($size - 1) \* $size; $key <= $size; $key += 1) {
    $array[$key] = 0;
}
 
$endTime = microtime(true);
 
echo $endTime - $startTime, ' seconds', "n";
{% endhighlight %}
我没亲自测过他们的速度差别，但耗时肯定不是一个数量级的。
再来看看Lua:
Luatable同时包含数组和哈希表部分，数值下标先在数组部分查找再到哈希表中查找。同PHP一样，Lua以哈希值取模分桶方式保存键值对，靠内部拉链解决冲突。哈希表部分初始分桶数为0（luaH_new@ltable.c:358），插入哈希表的键值对数量超过分桶数后，分桶数量倍增并将所有键值对重新哈希（rehash@ltable.c:333），数值、userdata、table等下标对(n-1)取模，字符串、boolean等下标对n取模（n为分桶数，总是2^k形式）（mainposition@ltable.c:100），数值下标（通常为double型）按32bit分为多段后累加得到哈希值（hashnum@ltable.c:84）。由于Lua中字符串是常量，故字符串下标创建时即计算出了对应的哈希值（luaS_newlstr@lstring.c:75）计算字符串哈希值时，并非字符串全部内容都参与计算，而是从后向前按步长k抽出字符计算，这里k=(L>>5)+1（这里给Hash冲突造成了可能），为节约存储空间，Lua将所有字符串加入全局字符串哈希表，新建字符串若已在全局表中存在则不再创建新实例。全局字符串表为分桶外部拉链结构，分桶数量按需倍增，按字符串哈希值对分桶数量取模进行分桶（同样这里也埋下了隐患）。
利用Lua字符串哈希值计算只挑选特定位置字符的特点，保证这些位置上的字符不变构造不同的字符串，使其哈希值完全相同，实现完全碰撞。
例如：固定字符串长度为32，保证从尾部向前每间隔1个字符都相同。
x00x11x00...x00x11x00
x00x22x00...x00x22x00
x00x33x00...x00x33x00
{% highlight lua %}
local t={}
local n=10000
fori=1,n do
    locals=(''):rep(28)..string.char(i/255)..''..string.char(i%255)..''
    --global string table colliding
    t[#t+1]=s
    --global string table and table hashcolliding
    --t[s]=1
end
{% endhighlight %}

如何防范Hash碰撞？

对于Asp.net，只需要打开自动更新即可，如果不能打开自动更新，则可更改web.config 或 applicationhost.config
{% highlight xml %}
<configuration>
  <system.web>
  <httpRuntime maxRequestLength="200”/>
  </system.web>
</configuration>
{% endhighlight %}

对于其他语言，尽快更新到最新版本如PHP 5.4有如下设置:
{% highlight php %}
      - max_input_vars - specifies how many GET/POST/COOKIE input variables may be
        accepted. default value 1000.
{% endhighlight %}
