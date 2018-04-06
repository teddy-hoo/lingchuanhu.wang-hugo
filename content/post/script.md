---
title: "Make your script robust"
date: 2018-04-05T22:02:05+08:00
draft: false
tags: ["实战经验"]
categories: ["系统设计"]
---

>以前在[Medium.com](https://medium.com/@teddyhoo/make-your-script-solid-3a1bd70359f2)上写的，懒得翻译直接搬过来了

# Make your script robust

In a web application, we often use a script in many cases, sync data to another application, do asynchronous jobs. Especially in a complicated application, the scripts take important roles. So, making your scripts solid help you avoiding risks, often complaints.

### 1. Monitoring
When the script starts? When the script ends? How long it takes? Does it works as scheduled? … You need know whether the script works as you expect. There are open source tools, like Azkaban(azkaban.github.io), with GUI, charts, help you know your scripts well.

### 2. Logging
This is another aspect to know how the scripts work. Not log the begin or end, log the details of the process. For example, a script which synchronizing the order data to data center. You should log, when the script starts, orders is read from database, orders synchronized to the data center successfully, orders synced failed. Otherwise, you should consider making the logs easy to read, easy to identify each script, easy to identify each execution.

---
The above two sections can help you targeting problems in your scripts. Never say that I do not need them. You WILL need them.

---

Imagine that your script failed because of network or whatever reason, the easiest way to fix it is RUN IT AGAIN, which means you need do nothing if you have scheduled your script.

### 3. Self-healing
In last execution, the script interrupted on an unexpected error. When the script runs, it should know the break point. Also the order syncing example, you can scan orders in ascending order on orderId filed or modifiedTime field or other increment field. When one sync task finished, script update the break point to Redis or other storage. When script runs again, gets the break point. I called this self-healing, the script can healing from a crash without humans’ help.

### 4. Re-entry
When the script crashed before record the break point, the last order will be synchronized twice when the script runs again. So you should make sure that the same task can be executed several times, but no effecting on the result.

### 5. NO breaking
Suppose 1000 orders need to be synced in this execution. An error occurred on 1st order, you should make sure that the script will continue executing until all orders tried instead of crashing. The error may be an illegal or an network break which you can predict when you writing your script.


---

Tips for optimizing your script.

---

### 6. Paralleling
When one instance of script execution can not met your performance requirements, you can run another instance in parallel. But when two instance executing, you should insure that they are not wasting compute resources. Also syncing orders case, two instance started from the same beak point, you got no performance improved. To solve this problem, you can use a distributed mutex lock, like Redis setnx command, mark each order as a mutex resource which is processing in your script.

### 7. Small piece of data
It is very common that millions rows of data in each execution of a script. It will be much better if you split them into small piece. A query consists of millions rows will consume lots of CPU time, in the meantime, CPU on sql server can not serve normally. It is not good case for a big system. Splitting them into small piece may increase count of query, but less sql server performance effect.

Hope helpful.