title: Linux日志查询命令journalctl使用手册
date: '2023-05-10 17:02:28'
updated: '2023-05-10 17:02:30'
tags:
  - linux
categories:
  - 每日一记
---
对于采用systemd管理服务的系统下，默认存在`systemd-journald`日志服务，并提供了`journalctl`命令查询记录的日志信息。
<!-- more -->

#### 常用命令

- 查询所有日志：`journalctl`
- 查询内核日志：`journalctl -k`
- 查询启动日志：`journalctl -b`
- 查询最近20条日志：`journalctl -n 20`
- 查询并跟踪日志：`journalctl -f`
- 查询冲突、告警、错误日志：`journalctl -p err..alert`
- 查询制定服务日志：`journalctl -u caddy.service -u nginx.service`
- 查询指定进程日志：`journalctl _PID=725`

#### 根据时间查询命令

- 查询20分钟前的日志：`journalctl --since "20 min ago"`
- 查询今天的日志：`journalctl --since today`
- 查询指定日期的日志：`journalctl --until 2023-05-01`