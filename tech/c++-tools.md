---
title:       "常见的c++性能调优工具"
subtitle:    ""
description: ""
date:        2024-05-08T14:53:59+08:00
author:      ""
image:       ""
tags:        ["c++", "工具"]
categories:  ["Tech" ]
---

## valgrind perf gporf

### 1 perf
Perf是内置于Linux内核源码树中的性能剖析工具。以性能事件为基础，常用于性能瓶颈的查找与热点代码的定位。

#### 1.1 使用
常见有两种使用方式：

* 直接使用perf启动二进制服务，有一个

```
perf record -e cpu-clock -g ./start
```
* 挂载到已启动的进程上
```
```

### 2 Gprof
gprof用于监控程序中每个方法的执行时间和被调用次数，方便找出程序中最耗时的函数。在程序正常退出后，会生成gmon.out文件，解析这个文件，可以生成一个可视化的报告





https://zhuanlan.zhihu.com/p/34629489