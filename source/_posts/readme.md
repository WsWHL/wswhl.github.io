title: This Blog README
date: '2020-04-05 10:06:41'
updated: '2023-05-09 14:23:13'
tags:
  - Blog
  - README
categories:
  - Deploy
---
## 介绍

本项目是使用Python语言开发的Blog网站。后端使用的是Django框架，前段使用的是Foundation JavaScript库。
仓库地址：https://github.com/WsWHL/blog
<!--more -->
## 环境
* 开发工具：vs code
* 数据库：my sql
* 缓存：redis
* 代理：nginx
> Python >= 3.7.2
> Django >= 2.1.6
> Foundation >= 6.5

## 开发
``` bash
export SECRET_KEY=******
python manage.py runserver 8000
```
* 开发工具vs code运行时会自动读取根目录下的`.env`环境变量文件，该文件中配置了必要的一些配置信息，不配置会使用默认值。出于安全考虑，`SECRET_KEY`只使用环境变量配置，该配置生成方式可参阅Django官方文档。

## 部署

- 系统：centos
- 服务：wsgi
- 代理：nginx
- 容器：docker

### 编译镜像
构建工具[docker-compose](https://docs.docker.com/compose/)
``` bash
docker-compose build
```
### 启动容器
``` bash
docker-compose up -d
docker-compose top # 查看运行中的容器状态
```
> -d 将进程放入后台执行

* 默认配置了ssl证书，相关配置参见根目录下的nginx文件夹中的nginx.conf配置文件。
