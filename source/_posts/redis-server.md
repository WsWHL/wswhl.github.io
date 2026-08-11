title: Redis服务
date: '2026-07-28 19:24:41'
updated: '2026-07-28 19:24:44'
tags:
  - Blog
categories:
  - 每日一记
---
1. 什么是Redis，Redis有哪些特点？
Redis全称为：Remote Dictionary Server（远程数据服务），Redis是一种支持key-value等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。
- 丰富的数据类型
- 内存存储：将所有的数据都存储在内存里面，数据读取和写入速度非常快
- 持久化功能：将数据存储在内存里面的数据保存到硬盘中，保证数据安全，方便进行数据备份和恢复。

2. Redis有哪些数据结构？
   - String：字符串
      `SET KEY_NAME VALUE`
   - Hash：哈希表
      `HSET KEY_NAME FIELD VALUE`
   - List：列表
      ```
      //在 key 对应 list 的头部添加字符串元素
        LPUSH KEY_NAME VALUE1.. VALUEN
        //在 key 对应 list 的尾部添加字符串元素
        RPUSH KEY_NAME VALUE1..VALUEN
        //对应 list 中删除 count 个和 value 相同的元素
        LREM KEY_NAME COUNT VALUE
        //返回 key 对应 list 的长度
        LLEN KEY_NAME 
      ```
   - Set：字典
      `SADD KEY_NAME VALUE1...VALUEn`
   - Sorted Set：有序集合
      `ZADD KEY_NAME SCORE1 VALUE1.. SCOREN VALUEN`

3. Redis如何做持久化的？及RDB和AOF的实现原理？

Redis是内存数据库，为了保证效率所有的操作都是在内存中完成。数据都是缓存在内存中，当你重启系统或者关闭系统，之前缓存在内存中的数据都会丢失再也不能找回。因此为了避免这种情况，Redis需要实现持久化将内存中的数据存储起来。

Redis官方提供了不同级别的持久化方式：
- RDB持久化：能够在指定的时间间隔能对你的数据进行快照存储。
- AOF持久化：记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾。Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
- 不使用持久化：如果你只希望你的数据在服务器运行的时候存在，你也可以选择不使用任何持久化方式。
- 同时开启RDB和AOF：你也可以同时开启两种持久化方式，在这种情况下当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。

RDB持久化：
RDB(Redis Database)持久化是把当前内存数据生成快照保存到硬盘的过程，触发RDB持久化过程分为手动触发和自动触发。

（1）手动触发
手动触发对应save命令，会阻塞当前Redis服务器，直到RDB过程完成为止，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。
（2）自动触发
自动触发对应bgsave命令，Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短。

在redis.conf配置文件中可以配置：
`save <seconds> <changes>`
 如果想关闭自动触发，可以在save命令后面加一个空串，即：
 `save ""`

AOF持久化：
AOF（append only file）持久化：以独立日志的方式记录每次写命令， 重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。
开启AOF功能需要配置：appendonly yes，默认不开启。

AOF文件名 通过appendfilename配置设置，默认文件名是appendonly.aof。保存路径同 RDB持久化方式一致，通过dir配置指定。

AOF的工作流程操作：
命令写入 （append）、文件同步（sync）、文件重写（rewrite）、重启加载 （load）。