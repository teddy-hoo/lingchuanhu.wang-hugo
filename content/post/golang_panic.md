---
title: "golang panic and recover"
date: 2018-04-02T22:32:05+08:00
draft: false
tags: ["语言设计坑"]
categories: ["编程语言"]
---

# golang panic and recover


## panic
Panic is a built-in function that stops the ordinary flow of control and begins panicking. When the function F calls panic, execution of F stops, any deferred functions in F are executed normally, and then F returns to its caller. To the caller, F then behaves like a call to panic. The process continues up the stack until all functions in the current goroutine have returned, at which point the program crashes. Panics can be initiated by invoking panic directly. They can also be caused by runtime errors, such as out-of-bounds array accesses.

panic简单理解就是golang runtime crash的方式，从函数调用一直往上抛，一直抛到进程崩溃。

看下面的例子：

```
func TestPanic(t *testing.T) {
	f := func() {
		panic("this is a panic!")
	}
	g := func() {
		f()
	}
	h := func() {
		t.Log("goroutine h ends!")
		time.Sleep(time.Second * 3)
	}
	t.Log("started!")
	go h()
	go g()
	time.Sleep(time.Second * 5)
}

#output 
panic: this is a panic!

goroutine 23 [running]:
tests.TestPanic.func1()
	/Users/wangwendy/tencent/pdmp-go-proxy/src/tests/utils_test.go:297 +0x39
tests.TestPanic.func2()
	/Users/wangwendy/tencent/pdmp-go-proxy/src/tests/utils_test.go:300 +0x24
created by tests.TestPanic
	/Users/wangwendy/tencent/pdmp-go-proxy/src/tests/utils_test.go:306 +0x8e
```

从上面的例子中可以看出，即使goroutine h还在运行中，但是g panic导致整个进程crash。

## recover

Recover is a built-in function that regains control of a panicking goroutine. Recover is only useful inside deferred functions. During normal execution, a call to recover will return nil and have no other effect. If the current goroutine is panicking, a call to recover will capture the value given to panic and resume normal execution.

recover的文档中有个重要的字眼是current goroutine，也就是说其他的goroutine之间使用recover就没有用了。

我们来看例子：

```
# 同一个goroutine
func TestRecover(t *testing.T) {
	f := func() {
		panic("this is a panic from f")
	}
	g := func() {
		defer func() {
			if r := recover(); r != nil {
				fmt.Println("Recovered message: ", r)
			}
		}()
		f()
	}
	g()

	time.Sleep(time.Second * 2)
	t.Log("normal exit")
}

# output
Recovered message:  this is a panic from f
	utils_test.go:295: normal exit
Process finished with exit code 0

# 不同的goroutine
func TestRecover(t *testing.T) {
	f := func() {
		panic("this is a panic from f")
	}
	g := func() {
		defer func() {
			if r := recover(); r != nil {
				fmt.Println("Recovered message: ", r)
			}
		}()
		go f()
	}
	g()

	time.Sleep(time.Second * 2)
	t.Log("normal exit")
}

#output 
panic: this is a panic from f

goroutine 21 [running]:
tests.TestRecover.func1()
	/Users/wangwendy/tencent/pdmp-go-proxy/src/tests/utils_test.go:282 +0x39
created by tests.TestRecover.func2
	/Users/wangwendy/tencent/pdmp-go-proxy/src/tests/utils_test.go:290 +0x58

Process finished with exit code 2
```

第一个例子中，f 和 recover在同一个goroutine中，panic被recover了，程序正常执行。
而不在一个goroutine时，即使f看上去是g的子协程，但是并不会被捕捉到。
