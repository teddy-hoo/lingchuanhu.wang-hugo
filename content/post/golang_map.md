---
title: "golang map内存使用分析"
date: 2018-05-20T17:02:05+08:00
draft: false
tags: ["底层理解"]
categories: ["编程语言"]
---
# golang map使用内存分析 
> golang提供了可以查看内存状态的[API](https://github.com/golang/go/blob/master/src/runtime/mstats.go#L159)如下代码

```go
func PrintMemUsage() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("Alloc = %v MiB", bToMb(m.Alloc))
    fmt.Printf("\tHeapAlloc = %v MiB", bToMb(m.HeapAlloc))
    fmt.Printf("\tTotalAlloc = %v MiB", bToMb(m.TotalAlloc))
    fmt.Printf("\tNumGC = %v\n", m.NumGC)
}
```

## 问题发现
看下面的代码

```go
func StringMap() {
    PrintMemUsage()
    y := make(map[string]string)
    for i := 0; i < 1000000; i++ {
        y[strconv.Itoa(i)] = strconv.Itoa(i)
        if (i + 1) % 100000 == 0 {
            fmt.Println("inserted ", i + 1, " items")
            PrintMemUsage()
        }
    }
}
```
输出为：

```
➜  awesomeProject go run main.go
Alloc = 0 MiB	HeapAlloc = 0 MiB	TotalAlloc = 0 MiB	NumGC = 0
inserted  100000  items
Alloc = 8 MiB	HeapAlloc = 8 MiB	TotalAlloc = 11 MiB	NumGC = 2
inserted  200000  items
Alloc = 17 MiB	HeapAlloc = 17 MiB	TotalAlloc = 22 MiB	NumGC = 3
inserted  300000  items
Alloc = 32 MiB	HeapAlloc = 32 MiB	TotalAlloc = 42 MiB	NumGC = 4
inserted  400000  items
Alloc = 35 MiB	HeapAlloc = 35 MiB	TotalAlloc = 46 MiB	NumGC = 4
inserted  500000  items
Alloc = 63 MiB	HeapAlloc = 63 MiB	TotalAlloc = 84 MiB	NumGC = 5
inserted  600000  items
Alloc = 65 MiB	HeapAlloc = 65 MiB	TotalAlloc = 86 MiB	NumGC = 5
inserted  700000  items
Alloc = 68 MiB	HeapAlloc = 68 MiB	TotalAlloc = 88 MiB	NumGC = 5
inserted  800000  items
Alloc = 72 MiB	HeapAlloc = 72 MiB	TotalAlloc = 92 MiB	NumGC = 5
inserted  900000  items
Alloc = 126 MiB	HeapAlloc = 126 MiB	TotalAlloc = 168 MiB	NumGC = 6
inserted  1000000  items
Alloc = 128 MiB	HeapAlloc = 128 MiB	TotalAlloc = 169 MiB	NumGC = 6
```
可以看到内存几乎一致在翻倍的增长，最大使用了128MB。代码中一共使用了2000000个String。
使用的内存大约为：

```python
// string 占用的空间
string space: 2000000 * 7 / 1024 / 1024 ~ 13.35 MB
// string 指针占用的内存，64bit的OS
string pointer space: 2000000 * 8 / 1024 / 1024 ~ 15.25 MB

total: 28.6 MB
```
这个结果与上面的输出相差了接近100MB，那么问题来了，这100MB哪里来的？

我们从以下几个方面分析。

# GC
[golang gc](http://lingchuanhu.wang/post/golang_gc/)与其他的语言的设计不同，他使用了几乎non-block的gc方法。所以上面的问题，我们首先想到的是没有充分GC。golang提供了主动GC的[接口](https://github.com/golang/go/blob/master/src/runtime/mgc.go#L1057)，这个是stop the world的gc。我们把代码做了如下改动：

```
func StringMap() {
    PrintMemUsage()
    y := make(map[string]string)
    for i := 0; i < 1000000; i++ {
        y[strconv.Itoa(i)] = strconv.Itoa(i)
        if (i + 1) % 100000 == 0 {
            fmt.Println("inserted ", i + 1, " items")
            PrintMemUsage()
            runtime.GC() // 主动GC，为了看中间过程，加不加对最后的结果没有影响
        }
    }
    runtime.GC() // 主动GC
    PrintMemUsage()
}
```
来看结果：
 
 ```
go run main.go
Alloc = 0 MiB	HeapAlloc = 0 MiB	TotalAlloc = 0 MiB	NumGC = 0
inserted  100000  items
Alloc = 8 MiB	HeapAlloc = 8 MiB	TotalAlloc = 11 MiB	NumGC = 2
inserted  200000  items
Alloc = 17 MiB	HeapAlloc = 17 MiB	TotalAlloc = 22 MiB	NumGC = 4
inserted  300000  items
Alloc = 32 MiB	HeapAlloc = 32 MiB	TotalAlloc = 42 MiB	NumGC = 6
inserted  400000  items
Alloc = 25 MiB	HeapAlloc = 25 MiB	TotalAlloc = 46 MiB	NumGC = 7
inserted  500000  items
Alloc = 63 MiB	HeapAlloc = 63 MiB	TotalAlloc = 84 MiB	NumGC = 9
inserted  600000  items
Alloc = 44 MiB	HeapAlloc = 44 MiB	TotalAlloc = 86 MiB	NumGC = 10
inserted  700000  items
Alloc = 47 MiB	HeapAlloc = 47 MiB	TotalAlloc = 88 MiB	NumGC = 11
inserted  800000  items
Alloc = 51 MiB	HeapAlloc = 51 MiB	TotalAlloc = 92 MiB	NumGC = 13
inserted  900000  items
Alloc = 126 MiB	HeapAlloc = 126 MiB	TotalAlloc = 168 MiB	NumGC = 15
inserted  1000000  items
Alloc = 125 MiB	HeapAlloc = 125 MiB	TotalAlloc = 169 MiB	NumGC = 16
Alloc = 87 MiB	HeapAlloc = 87 MiB	TotalAlloc = 169 MiB	NumGC = 18
 ```
 GC是有效果的，减少了41MB内存。
 
# HashMap Overload Factor

### load factor分析
load factor [这里](http://javabypatel.blogspot.hk/2015/10/what-is-load-factor-and-rehashing-in-hashmap.html)有个解释，简单讲就是在什么时候扩容hashmap的bucket size，是空间和时间的权衡，查了很多资料一般都会使用0.75，就是当bucket的空间使用了75%时，double size bucket。[golang 也使用了](https://github.com/golang/go/blob/master/src/runtime/map.go#L998)相同的算法。load factor 0.75能推导出来一下：

```
假如实际空间要使用75MB，那么hashmap
最少使用内存：75 / 0.75 = 100 MB
最多使用内存：75 / 0.75 * 2 = 200MB

所以插入1000000个item
最少使用内存：1000000 / 0.75 * bucket size
最多使用内存：1000000/ 0.75 * 2 * bucket size
```

### 修改golang源码 打日志验证
为了验证这个理论，对golang的源码做了如下改动。

```
// https://github.com/golang/go/blob/master/src/runtime/map.go#L954
// 打印出了bucket double size的信息
func hashGrow(t *maptype, h *hmap) {
	// ...
	oldbuckets := h.buckets
	newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)
	print("old bucket length: ", h.B, "\n")  // 扩容前的大小
	print("new bucket length: ", h.B + bigger, "\n") // 扩容后的大小
	print("bucket size: ", t.bucketsize, "\n") // 每个bucket的大小
}
```
输出结果：

```
old bucket length: 0
new bucket length: 1
bucket size: 272
old bucket length: 1
new bucket length: 2
bucket size: 272
old bucket length: 2
new bucket length: 3
bucket size: 272
...
inserted  100000  items
Alloc = 8 MiB	HeapAlloc = 8 MiB	TotalAlloc = 11 MiB	NumGC = 2
old bucket length: 14
new bucket length: 15
bucket size: 272
inserted  200000  items
Alloc = 17 MiB	HeapAlloc = 17 MiB	TotalAlloc = 22 MiB	NumGC = 4
old bucket length: 15
new bucket length: 16
bucket size: 272
inserted  300000  items
Alloc = 32 MiB	HeapAlloc = 32 MiB	TotalAlloc = 42 MiB	NumGC = 6
inserted  400000  items
Alloc = 25 MiB	HeapAlloc = 25 MiB	TotalAlloc = 46 MiB	NumGC = 7
old bucket length: 16
new bucket length: 17
bucket size: 272
inserted  500000  items
Alloc = 63 MiB	HeapAlloc = 63 MiB	TotalAlloc = 84 MiB	NumGC = 9
inserted  600000  items
Alloc = 44 MiB	HeapAlloc = 44 MiB	TotalAlloc = 86 MiB	NumGC = 10
inserted  700000  items
Alloc = 47 MiB	HeapAlloc = 47 MiB	TotalAlloc = 88 MiB	NumGC = 11
inserted  800000  items
Alloc = 51 MiB	HeapAlloc = 51 MiB	TotalAlloc = 92 MiB	NumGC = 13
old bucket length: 17
new bucket length: 18
bucket size: 272
inserted  900000  items
Alloc = 126 MiB	HeapAlloc = 126 MiB	TotalAlloc = 168 MiB	NumGC = 15
inserted  1000000  items
Alloc = 125 MiB	HeapAlloc = 125 MiB	TotalAlloc = 169 MiB	NumGC = 16
Alloc = 87 MiB	HeapAlloc = 87 MiB	TotalAlloc = 169 MiB	NumGC = 18

```

>  下面的运算中都会用到一个8  这个代表每个bucket最多存储的key的个数 源码见：// 8的来源 https://github.com/golang/go/blob/master/src/runtime/map.go#L63

先看内存使用是否对的上：

```
将bucket size套入上面的公式
min: 1000000 / 8 / 0.75 * 272 =  43.2 MB
max: 1000000 / 8 / 0.75 * 2 * 272 = 86.4 MB

看最后一次grow
272 * 2^18  = 68 MB
```
在推断的区间内。

再看load factor是否符合，为了看的更清楚，我每隔10000次插入打印一次，输出的片段结果：

```
...
inserted  850000  items
Alloc = 53 MiB	HeapAlloc = 53 MiB	TotalAlloc = 95 MiB	NumGC = 88
old bucket length: 17
new bucket length: 18
bucket size: 272
inserted  860000  items
Alloc = 126 MiB	HeapAlloc = 126 MiB	TotalAlloc = 167 MiB	NumGC = 90
...
```
在大约850000个item的时候，进行了扩容：

```
850000 / (2^17 * 8) = 81%
```
基本符合，为什么不是完全一致，这里简单分析一下，因为hash value冲突的时候，golang的实现了使用的是Chaining（链地址法）算法。误差来源自一共产生了多少次冲突，并且有多少冲突在bucket内没有存储完毕，使用了chaining。chaining会申请额外的内存记录在bucket中，所以内存上会有出入。

### 为什么最大使用内存到了122 MB
还是日志片段：

```
inserted  850000  items
Alloc = 53 MiB	HeapAlloc = 53 MiB	TotalAlloc = 95 MiB	NumGC = 88
old bucket length: 17
new bucket length: 18
bucket size: 272
inserted  860000  items
Alloc = 126 MiB	HeapAlloc = 126 MiB	TotalAlloc = 167 MiB	NumGC = 90
...
inserted  950000  items
Alloc = 122 MiB	HeapAlloc = 122 MiB	TotalAlloc = 169 MiB	NumGC = 99
inserted  960000  items
Alloc = 86 MiB	HeapAlloc = 86 MiB	TotalAlloc = 169 MiB	NumGC = 100
...
```
上面的输出中还有一点值得注意，为什么grow到2^18个bucket之后，使用的内存是128MB，又过了一段时间之后，内存才降到86MB

这是因为在hashGrow之后，并不会直接做memcopy，而是每次遇到插入新的item的时候，对涉及到的bucket copy到新的bucket空间内，直到所有的旧的bucket里面的内容全部copy新的bucket的时候，才会把旧的bucket空间释放。[源码见](https://github.com/golang/go/blob/master/src/runtime/map.go#L578)。

所以之前的分析的最大使用内存应该是：

```
item_count * bucket_size + item_count / 0.75 * 2 * bucket_size
```

# 为什么是86MB

### map存的是指针
前面的日志显示，最后一次grow完之后，map使用的空间应该是68MB，但是结果为什么是86MB。

看一下map的结构体：

```
type maptype struct {
	typ           _type
	key           *_type
	elem          *_type
	bucket        *_type // internal type representing a hash bucket
	keysize       uint8  // size of key slot
	indirectkey   bool   // store ptr to key instead of key itself
	valuesize     uint8  // size of value slot
	indirectvalue bool   // store ptr to value instead of value itself
	bucketsize    uint16 // size of bucket
	reflexivekey  bool   // true if k==k for all keys
	needkeyupdate bool   // true if we need to update key on an overwrite
}
```

里面有两个值，indirectkey和indirectvalue，这里代表bucket里面存的是数据的指针还是数据本身。可想而知，key和value都是字符串，存的肯定是指针，同样做了验证。

```
// https://github.com/golang/go/blob/master/src/runtime/map_faststr.go#L191

print("insert new key: ", t.key, " value: ", t.elem, "\n")
print("key size: ", t.keysize, " value size: ", t.valuesize, "\n")
print("direct key: ", t.indirectkey, "direct value: ", t.indirectvalue, "\n")
	
direct key: falsedirect value: false
insert new key: 0x10a1460 value: 0x10a1460
key size: 16 value size: 16
```

所以2000000个string并没有存在map里面，而是存在额外的heap空间里。

### 向操作系统申请的heap空间一般情况下不会全部被使用

应用程序向操作系统申请的heap space，会存在一些内存碎片，不会完全使用。在这里不展开分析。

# 总结

对一些内存比较敏感的应用程序来说，这样的分析还是必要的，至少可以搞清楚以下几个问题：

1. 数据结构的使用方法错了？
2. 数据结果选择错了？
3. 算法是不是该改正一下？
4. 一台32GB内存的机器 可以承载什么样的业务体量 （物理内存被用完了，CPU效率会急剧下降）