---
title: "golang slice"
date: 2018-03-01T13:02:05+08:00
draft: false
---

# golang slice append

```
func TestSliceAppend(t *testing.T) {
    a := []int{1,2,3}
    fmt.Println("a:\t\t\t", a)
    fmt.Println("len(a):\t\t\t", len(a))
    fmt.Println("cap(a):\t\t\t", cap(a))
    a = []int{1,2,3}
    b := append(a, 4)
    fmt.Println("a:\t\t\t", a)
    fmt.Println("len(a):\t\t\t", len(a))
    fmt.Println("cap(a):\t\t\t", cap(a))
    fmt.Println("b:\t\t\t", b)
    fmt.Println("len(b):\t\t\t", len(b))
    fmt.Println("cap(b):\t\t\t", cap(b))
    a = []int{1,2,3}
    c := append(a[:2], 5)
    fmt.Println("a:\t\t\t", a)
    fmt.Println("len(a):\t\t\t", len(a))
    fmt.Println("cap(a):\t\t\t", cap(a))
    fmt.Println("c:\t\t\t", c)
    fmt.Println("len(c):\t\t\t", len(c))
    fmt.Println("cap(c):\t\t\t", cap(c))
    a = []int{1,2,3}
    d := append(a[:2], []int{6,7,8}...)
    fmt.Println("a:\t\t\t", a)
    fmt.Println("len(a):\t\t\t", len(a))
    fmt.Println("cap(a):\t\t\t", cap(a))
    fmt.Println("d:\t\t\t", d)
    fmt.Println("len(d):\t\t\t", len(d))
    fmt.Println("cap(d):\t\t\t", cap(d))
}

a:			 [1 2 3]
len(a):			 3
cap(a):			 3
a:			 [1 2 3]
len(a):			 3
cap(a):			 3
b:			 [1 2 3 4]
len(b):			 4
cap(b):			 6
a:			 [1 2 5]
len(a):			 3
cap(a):			 3
c:			 [1 2 5]
len(c):			 3
cap(c):			 3
a:			 [1 2 3]
len(a):			 3
cap(a):			 3
d:			 [1 2 6 7 8]
len(d):			 5
cap(d):			 6
```
上面的例子中比较奇怪的是，有的`append`改变了原数组的值，有的没有改变原数组的值。
先看一下，golang中slice的数据结构：
```
type slice struct{
	int len,
	int cap,
	*T p
}
```
其中p指向一个c语言中的数组，len表示当前slice的长度，而cap表示p指向的数组的长度。

当执行`append`操作时，如果当前数组cap满足需求，则不会申请新的数组，`append`会创建新的slice对象，但是p会指向原来的数组，这就解释了`append(a[:2], 5)`的结果。

如果当前数组的cap不满足`append`的需求时，会创建新的数组，`append`返回新的slice中的p会指向新的数组，这里解释`append(a[:2], [6,7,8])`的结果。
