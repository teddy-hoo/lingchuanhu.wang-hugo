---
title: "do not leave echo in your php code"
date: 2017-04-25T13:02:05+08:00
draft: false
tags: ["最佳实践"]
categories: ["编程语言"]
---

# do not leave echo in your php code

As we know, echo statement has bad performance. The following is a test for echo performance.

Codes
```
<?php
# test-echo.php
fclose(STDOUT);
$STDOUT = fopen('./output', 'wb');
$i = 0;
while ($i < 10000000) {
    $i += 1;
    echo 'hello world';
}
<?php
# test.php
$i = 0;
while ($i < 10000000) {
    $i += 1;
    $x = rand(1000000, 9999999);
}
```
Benchmark
```
$ time php test-echo.php
real    0m10.693s
user    0m2.111s
sys     0m8.436s
$ time php test.php
real    0m5.613s
user    0m4.908s
sys     0m0.686s
```


CPU Info
```
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 42
model name      : Intel Xeon E312xx (Sandy Bridge)
stepping        : 1
microcode       : 0x1
cpu MHz         : 3292.518
cache size      : 4096 KB
siblings        : 4
```