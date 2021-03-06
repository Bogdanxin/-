[TOC]

# InnoDB引擎

## 1.InnoDB体系架构

![202004101526](https://i.loli.net/2020/04/10/7CSFq4iJRYPk9ox.png)

* 维护多有进程/线程需要访问的多个内部数据结构

* 缓存磁盘上的数据，方便快速地读取，同时在对磁盘文件的数据修改之前在这里缓存

* 重做日志（redo log）缓冲

  ...

### 1.1 后台线程

InnoDB储存引擎是多线程的模型，有多个线程负责不同任务

1. Master Thread 

   Master Thread 是核心后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据一致性。

2. IO Thread 

   IO Thread 主要用来处理储存引擎中