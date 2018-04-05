---
title: "golang slice"
date: 2018-03-01T13:02:05+08:00
draft: false
tags: ["语言设计坑"]
categories: ["编程语言"]
---

# golang slice append

```
func TestSliceAppend(t *testing.T) {
    a := []int{1,2,3}
    fmt.Println("a:\t\t\t", a)             // [1 2 3]
    fmt.Println("len(a):\t\t\t", len(a))   // 3
    fmt.Println("cap(a):\t\t\t", cap(a))   // 3
    a = []int{1,2,3}
    b := append(a, 4)
    fmt.Println("a:\t\t\t", a)             // [1 2 3]
    fmt.Println("len(a):\t\t\t", len(a))   // 3
    fmt.Println("cap(a):\t\t\t", cap(a))   // 3
    fmt.Println("b:\t\t\t", b)             // [1 2 3 4]
    fmt.Println("len(b):\t\t\t", len(b))   // 4
    fmt.Println("cap(b):\t\t\t", cap(b))   // 6
    a = []int{1,2,3}
    c := append(a[:2], 5)
    fmt.Println("a:\t\t\t", a)             // [1 2 5]
    fmt.Println("len(a):\t\t\t", len(a))   // 3
    fmt.Println("cap(a):\t\t\t", cap(a))   // 3
    fmt.Println("c:\t\t\t", c)             // [1 2 5]
    fmt.Println("len(c):\t\t\t", len(c))   // 3
    fmt.Println("cap(c):\t\t\t", cap(c))   // 3
    a = []int{1,2,3}                    
    d := append(a[:2], []int{6,7,8}...)
    fmt.Println("a:\t\t\t", a)             // [1 2 3]
    fmt.Println("len(a):\t\t\t", len(a))   // 3
    fmt.Println("cap(a):\t\t\t", cap(a))   // 3
    fmt.Println("d:\t\t\t", d)             // [ 1 2 6 7 8]
    fmt.Println("len(d):\t\t\t", len(d))   // 5
    fmt.Println("cap(d):\t\t\t", cap(d))   // 6
}
```

先列出了一些例子，`c := append(a[:2], 5)`这一行改变了slice a，但是`d := append(a[:2], []int{6,7,8}...)`这个一行又没有改变slice a，不太好解释，也不太明白，这里的append到底做了什么。

先看一下，golang中slice的数据结构：
```
type slice struct{
	int len,
	int cap,
	*T p
}

// 存储方式

+------+
| len  |
+------+
| cap  |
+------+         +-----+-----+-----+
|  p   |  -----> |  1  |  2  |  3  |
+------+         +-----+-----+-----+
```
其中p指向一个c语言中的数组，这个数组是的长度是固定的，len表示当前slice的长度，而cap表示p指向的数组的长度。

当执行`append`操作时，如果当前数组cap满足需求，则不会申请新的数组，`append`会创建新的slice对象，但是p会指向原来的数组，这就解释了`append(a[:2], 5)`的结果。

```
+------+
| len  |
+------+
| cap  |
+------+         +-----+-----+-----+
|  p   |  -----> |  1  |  2  |  5  |
+------+         +-----+-----+-----+
```

如果当前数组的cap不满足`append`的需求时，会创建新的数组，`append`返回新的slice中的p会指向新的数组，这里解释`append(a[:2], [6,7,8])`的结果。

```
+------+
| len  |
+------+
| cap  |
+------+         +-----+-----+-----+-----+-----+-----+
|  p   |  -----> |  1  |  2  |  6  |  7  |  8  |  0  |
+------+         +-----+-----+-----+-----+-----+-----+
```

>append 的文档 from golang.org
The append built-in function appends elements to the end of a slice. If it has sufficient capacity, the destination is resliced to accommodate the new elements. If it does not, a new underlying array will be allocated. Append returns the updated slice. It is therefore necessary to store the result of append, often in the variable holding the slice itself。

其实文档里也说明了这个问题，但是`Append returns the updated slice`这句话着实有些迷惑，是update了原来的slice呢，还是update到了新的slice呢。还是要实验。

有意思的是，如果我想`append(a[:2], 5)`得到这个结果，改怎么办呢？

```
var a []int
a = append(a, a[:2]...,)
a = append(a, 5)
```


