title: 解决centos dnf自动更新异常问题
date: '2023-05-10 16:09:38'
updated: '2023-05-10 16:09:41'
tags:
  - linux
categories:
  - 每日一记
---
#### 起因

最近发现代理服务总是出现莫名中断不可访问的异常问题，刚开始怀疑是机器搭建的服务太多导致内存占用过高导致的，于是用top命令查了一下占用最多的服务也就200Mb，总共占用也就百分之六七十，不应该导致服务崩溃，于是又查询了一下系统日志才发现端倪.

#### 问题排查

查询系统错误和警告日志：
```shell
journalctl -p err..alert
```
> kernel: Out of memory: Killed process 1813 (dnf) total-vm:842700kB, anon-rss:259464kB, file-rss:0kB, shmem-rss:0kB, UID:0 pgtables:1236kB oom_score_adj:0
> systemd[1]: Failed to start dnf makecache.

*原来是由于dnf软件包信息更新导致的系统OOM崩溃.*

#### 解决方案

由于主机内存有限（1G）, 于是只能选择先禁用，优先保证服务正常运转.

禁用定时服务：
```shell
systemctl stop dnf-makecache.timer
systemctl disable dnf-makecache.timer
```