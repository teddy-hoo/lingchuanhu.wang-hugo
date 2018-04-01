---
title: "golang slice"
date: 2018-03-01T13:02:05+08:00
draft: false
---

golang slice append

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



```golang
func BenchmarkSliceAppend(b *testing.B) {
    a := make([]int, 0, b.N)
    for i := 0; i < b.N; i++ {
        a = append(a, i)
        if i % 1000000 == 0 {
            fmt.Println("cap(a)", cap(a))
        }

    }
}
/*
cap(a) 1
goos: darwin
goarch: amd64
pkg: tests
cap(a) 100
cap(a) 10000
cap(a) 1000000
cap(a) 100000000
...
100000000	        10.3 ns/op
PASS
*/

func BenchmarkSliceAppendDynamic(b *testing.B) {
    a := make([]int, 0)
    for i := 0; i < b.N; i++ {
        a = append(a, i)
        if i % 1000000 == 0 {
            fmt.Println("cap(a)", cap(a))
        }
    }
}
/*
cap(a) 1
...
cap(a) 1136640
cap(a) 2221056
cap(a) 3471360
cap(a) 4339712
cap(a) 5425152
cap(a) 6781952
...
cap(a) 10597376
cap(a) 10597376
cap(a) 13247488
...
cap(a) 16560128
...
cap(a) 20700160
...
cap(a) 1
cap(a) 1136640
cap(a) 2221056
cap(a) 3471360
cap(a) 4339712
cap(a) 5425152
cap(a) 6781952
cap(a) 8477696
...
cap(a) 10597376
...
cap(a) 13247488
...
cap(a) 16560128
...
cap(a) 20700160
...
cap(a) 25875456
...
cap(a) 32345088
...
cap(a) 40431616
...
cap(a) 50539520
...
cap(a) 63174656
...
cap(a) 78968832
...
cap(a) 98711552
...
cap(a) 123389952
100000000	        32.1 ns/op
PASS
*/
```
