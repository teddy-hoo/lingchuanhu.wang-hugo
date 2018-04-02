---
title: "Select IO 多路复用"
date: 2018-04-02T22:02:05+08:00
draft: false
tags: ["基础知识"]
categories: ["操作系统"]
---

[示例代码](https://github.com/teddy-hoo/computer-systems-a-programmer-s-perspective/blob/master/code/select.cpp)

select的功能是，能够监听多个IO，以达到一个进程内能够同时处理多个系统IO。

#### 以网络监听为需求描述：
1. 当启动一个网络IO时，进程会阻塞在listen函数调用，当有链接请求时，进程继续执行；
2. 当这个进程有了已经建立的链接时，就要阻塞在read函数，来读取客户端发送的内容；
3. 当进程阻塞在read函数时，就无法处理listen函数已经准备好的IO，即无法接受新的链接请求；
4. 准确的说，此时进程有两个系统IO在等待，但是进程不能同时处理两个IO；
5. select函数就应运而生，他可以同时去等待多个IO事件，其中一个或者多个IO准备好的时候，唤醒进程以处理IO；

#### select的工作过程
1. 将需要等待的fd传给select，并且传入一个全部为0的fd标志位；
2. 当有IO ready时，select会返回；
3. 通过遍历不为0的fd标志位来得到到底哪些IO时间ready以处理；

