---
title: "golang gc"
date: 2018-04-03T23:03:05+08:00
draft: false
tags: ["底层理解"]
categories: ["编程语言"]
---

> https://making.pusher.com/golangs-real-time-gc-in-theory-and-practice/
这篇文章图解了tri-color collector，非常好。从原理上讲解了golang如何做到concurrency gc。

> https://blog.golang.org/go15gc

```
package main

import (
	"fmt"
	"time"
)

const (
	windowSize = 200000
	msgCount   = 1000000
)

type (
	message []byte
	buffer  [windowSize]message
)

var worst time.Duration

func mkMessage(n int) message {
	m := make(message, 1024)
	for i := range m {
		m[i] = byte(n)
	}
	return m
}

func pushMsg(b *buffer, highID int) {
	start := time.Now()
	m := mkMessage(highID)
	(*b)[highID%windowSize] = m
	elapsed := time.Since(start)
	if elapsed > worst {
		worst = elapsed
	}
}

func main() {
	var b buffer
	for i := 0; i < msgCount; i++ {
		pushMsg(&b, i)
	}
	fmt.Println("Worst push time: ", worst)
}
```