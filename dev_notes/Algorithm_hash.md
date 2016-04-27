# HASHING: THE BASICS

## 问题描述

搜索引擎会通过日志文件把用户每次检索使用的所有检索串都记录下来，每个查询串的长度为1-255字节。假设目前有一千万个记录（这些查询串的重复度比较高，虽然总数是1千万，但如果除去重复后，不超过3百万个。一个查询串的重复度越高，说明查询它的用户越多，也就是越热门。），请你统计最热门的10个查询串，要求使用的内存不能超过1G。


## 什么是哈希表？
哈希表（Hash table，也叫散列表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

哈希表hashtable(key，value) 的做法其实很简单，就是把Key通过一个固定的算法函数既所谓的哈希函数转换成一个整型数字，然后就将该数字对数组长度进行取余，取余结果就当作数组的下标，将value存储在以该数字为下标的数组空间里。

而当使用哈希表进行查询的时候，就是再次使用哈希函数将key转换为对应的数组下标，并定位到该空间获取value，如此一来，就可以充分利用到数组的定位性能进行数据定位

哈希表适用于 有大量查找的场合。

## 问题解析：

要统计最热门查询，首先就是要统计每个Query出现的次数，然后根据统计结果，找出Top 10。所以我们可以基于这个思路分两步来设计该算法。

 - 第一步：Query统计
    Query统计有以下俩个方法，可供选择：
    1. 直接排序法
        - 一千万条记录，每条记录是255Byte，很显然要占据2.375G内存，直接排序无法满足内存需求
        - 当数据量比较大而且内存无法装下的时候，我们可以采用外排序的方法来进行排序，这里我们可以采用归并排序。
        - 排完序之后我们再对已经有序的Query文件进行遍历，统计每个Query出现的次数，再次写入文件中。
    2. Hash Table法
        - 维护一个Key为Query字串，Value为该Query出现次数的HashTable 
 - 第二步：找出Top 10
    - 算法一：普通排序
    - 算法二：堆排序

## Operations and Applications

**Purpose**: maintain a (possibly evolving) set of stuff. (transactions, people associated data, IP address, etc. )

**Insert**: add new record

**Delete**: delete existing record

**Lookup**: check for a particular record

A hash table does not maintain an ordering on the elements that it supports.

All operations in O(1) time! 


**Applications**:

 - de-duplication  处理重复条目
 - the 2-SUM problem 
    - Input: unsorted array A  of n integers  
    - determine whether or not there are two numbers x,y = t
    - Naive solution: O(n²)
    - Better :  (1) sort A ( O(nlogn) ) .  (2) for each x in A , look t-x in A via binary seach ( O(nlogn) )
    - Amazing ： (1） insert elements into hash table H ,O(n).  (2) for each x in A, look up t-x in H , O(n)
 - symbol tables in compilers
 - black list
 - search algorithms , eg. in a chess playing program , the game tree exploration
    - use hash table to avoid exploring any configuration more than once.

## Hash Tables: Implementation Details, Part I

### High Idea

 1. pick n= #buckets , with n ≈ |S|
 2. choose a hash function :
    - input key -> hash function -> position in array [0,1,2,...,n-1]
 3. use array A of length n, store x in A[h(x)]

23个人中，有两个人生日同天的概率 >= 50%.

所以，hash 出现 collision 的概率是非常高的。

### Resolving Collisions

Collision: distinct x,y∈U, such that h(x)=h(y)

 - Solution#1: separate chaining
    - keep a link list in each bucket
    - 1 -> A
    - 2 -> nil
    - 3 -> B -> D
    - 4 -> C
 - Solution#2: open addressing
    - only 1 object per bucket
    - hash function replaced by probe sequence hs₁(x),h₂(x),...
    
哈希表有多种不同的实现方法，最常用的一种方法——拉链法，我们可以理解为“链表的数组”

![](http://hi.csdn.net/attachment/201103/17/8394323_1300353335qQMM.jpg)

左边很明显是个数组，数组的每个成员包括一个指针，指向一个链表的头，当然这个链表可能为空，也可能元素很多。我们根据元素的一些特征把元素分配到不同的链表中去，也是根据这些特征，找到正确的链表，再从链表中找出这个元素。

元素特征转变为数组下标的方法就是散列法。散列法当然不止一种，下面列出三种比较常用的：

 1. 除法散列法 
    - 最直观的一种，上图使用的就是这种散列法，
    - 公式：index = value % 16  
 2. 平方散列法 
    - 求index是非常频繁的操作，而乘法的运算要比除法来得省时（对现在的CPU来说，估计我们感觉不出来），所以我们考虑把除法换成乘法和一个位移操作。 
    - 公式：index = (value * value) >> 28  
    - 如果数值分配比较均匀的话这种方法能得到不错的结果，但我上面画的那个图的各个元素的值算出来的index都是0——非常失败。也许你还有个问题，value如果很大，value * value不会溢出吗？答案是会的，但我们这个乘法不关心溢出，因为我们根本不是为了获取相乘结果，而是为了获取index。
 3. 斐波那契（Fibonacci）散列法
    - 平方散列法的缺点是显而易见的，所以我们能不能找出一个理想的乘数，而不是拿value本身当作乘数呢？答案是肯定的。
        1. 对于16位整数而言，这个乘数是40503 
        2. 对于32位整数而言，这个乘数是2654435769 
        3. 对于64位整数而言，这个乘数是11400714819323198485
    - 这几个“理想乘数”是如何得出来的呢？这跟一个法则有关，叫黄金分割法则，而描述黄金分割法则的最经典表达式无疑就是著名的斐波那契数列，即如此形式的序列：0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610， 987, 1597, 2584, 4181, 6765, 10946，…
    -  对我们常见的32位整数而言
    -  公式：index = (value * 2654435769) >> 28

如果用这种斐波那契散列法的话，那上面的图就变成这样了：

![](http://hi.csdn.net/attachment/201103/17/8394323_130035338777Q4.jpg)

### 适用范围

快速查找，删除的基本数据结构，通常需要总数据量可以放入内存。

### 基本原理及要点

hash函数选择，针对字符串，整数，排列，具体相应的hash方法。 
碰撞处理，一种是open hashing，也称为拉链法；另一种就是closed hashing，也称开地址法，opened addressing。


 
## Hash Tables: Implementation Details, Part II

 -  ***Note***: in hash table with chaining, Insert is θ(1)
    - insert new object x at front of list in A[h(x)]
    - O( list length ) for Insert / Delete
 - ***Point***: performance depends on the choice of hash function
 - Properties of a *Good* Has Function:
    1. should lead to good performance 
        - ie, should "spread data out" 
        - gold standard: completely random hashing
    2. should be easy to store / very fast to evaluated

### Bad Hash Functions

06:57

# Universal Hashing

TODO

# BLOOM FILTERS (Week 6)

## 提出问题

Google的爬虫每天需要抓取大量的网页。于是就有一个问题：每当爬虫分析出一个url的时候，是抓呢，还是不抓呢？如何知道这个url已经爬过了？

这个问题，归纳抽象后可以定义为：

给定一个集合S（注意，这里的集合是传统意义上的集合：元素彼此不同。本文不考虑multiset），给定一个元素e，需要判断e∈S 是否成立。（学术界一般称为membership问题）

## 分析问题

都有哪些方案可以解决这个问题？

一种简单的想法是把url存储在一个哈希表中，每次去表里look up下判断是否存在。假如每个url占用40B，那么10亿条url将占用大概30多GB的内存！Can this be more space efficient ?

## 解决问题

我们可不可以不存url本身？这样子所需空间就会大大减少了。于是我们想到一个很经典的做法：bitmap（位图）。将集合S中的url哈希到bitmap上，给定一个url，只需要将它hash，得到它在bitmap的下标，检查该位置是否为1即可。

这样做空间是省了，可是也产生了一个问题：由于冲突（碰撞），不是集合S中的元素也可能被哈希到值为1的位置上，导致误报。

给定一个元素e，如果实际上e∉S  而被判为 e∈S ，那么我们称e是false positive（伪正例。顺便说一句，false positive等的分析在machine learning的classification任务里评价model时非常重要）。

如何降低false positive的概率呢？Bloom Filter的想法是使用多个独立的哈希函数。

## Standard Bloom Filter

在传统的Bloom Filter中，我们有：

 - 集合S：其大小为m。也就是说，集合中有m个不同元素。
 - 可用内存B：B被当成位数组bitmap来使用，大小为n。（有n个bit）。
 -哈希函数：有k个独立的、均匀分布的哈希函数。

Bloom Filter的做法是：初始时，所有比特位都初始化为0。对于集合中的每个元素，利用k个哈希函数，对它哈希得到k个位置，将bitmap中的对应的k个位置置为1。

给定一个元素e，为了判断它是否是集合中的元素，也利用该k个函数得到k个位置，检查该k个位置是否都为1，如果是，认为e∈S ，否则认为e∉S 。

不难看出，如果e∈S ，那么Bloom Filter肯定会正确判断出e∈S ，但是它还是可能产生false positive。那么，如何分析false positive的概率呢？

当n和m固定时，选择k=n/m·ln2  附近的一个整数，将使false positive possibility最小。

工程实现时，我们需要k个哈希函数或者哈希函数值。如何构造和获得k个独立的哈希函数呢？ 其实只需要两个独立的哈希函数hf1和hf2即可，也就是通过如下方式获得k个哈希函数值：

```
hash value = hf1(key) + i*hf2(key)
```

其中i=0、1、2…k-1


## Bloom Filters: the basic

more space efficient

 1. can not store an associated object
 2. no deletion
 3. small false postive probability 
    - can actually make mistakes
    - might say x has been inserted even though it hasn't been

### Application

 - Original: early spellchecker
 - Canonical: list of forbidden passwords
 - Modern: network routers
    - limit memory need to be super-fast

### Under the hood

Ingredient: 
 
- array of n bits
- k hash functions: h₁,h₂,...,hk
    - k = small constant

Inserti(x):

```
for i=1,2,...,k:
    # whether or not bit already set to 1
    set A[hi(x)] = 1
```

Lookup(x):

```
return True <=> A[hi(x)] = 1 for every i=1,2,...,k
```

Note: 

 - no false negatives
    - if x was inserted, lookup(x) guaranteed to success
 - but false postive
    - if all k hi(x) is already set to 1 by other insertions.

## Bloom Filters: Heuristic Analysis 
    
TODO