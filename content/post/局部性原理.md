---
title: "Local"
date: 2018-04-01T13:02:05+08:00
draft: false
---

# 局部性原理

计算机程序倾向于引用邻近于其他最近引用过得数据项的数据项，或者最近引用过的数据项本身。

局部性性原理是一个持久的概念，对硬件和软件系统的设计和性能都有着极大的影响。

### 时间局部性 temporal locality

在一个具有良好的时间局部性的程序中，被引用过一次的存储器位置很可能在不久的将来再被多次引用。

### 空间局部性 spatial locality

在一个具有良好空间局部性的程序汇总，如果一个存储器位置被引用了一次，那么程序很可能在不远的将来引用附近的一个存储器位置。

## 对程序数据引用的局部性

步长为一的引用模式比较契合局部性原理。

```
#include<stdio.h>

int sum_array_cols(int a[][1000], int m, int n)
{
        int i, j, sum = 0;
        for (i = 0; i < m; i ++)
                for (j = 0; j < n; j++) {
                        sum = sum + a[i][j];
                }
        return sum;
}

int sum_array_cols_1(int a[][1000], int m, int n)
{
        int i, j, sum = 0;
        for (j = 0; j < n; j++)
                for (i = 0; i < m; i++)
                        sum += a[i][j];
        return sum;
}


int main()
{
        int m = 2000;
        int n = 1000;
        int a[m][n];
        int i, j;
        for (i = 0; i < m; i ++)
                for (j = 0; j < n; j ++)
                        a[i][j] = 1;
        int x = sum_array_cols(a, m, n);
        printf("%d\n", x);
}
```



