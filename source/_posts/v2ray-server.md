---
title: v2ray工具配置教程 
date: '2020-04-05 14:14:57'
updated: '2020-04-05 14:14:57'
categories:
  - 科学上网
tags:
  - V2ray
  - Proxy
---
- 常说授人以鱼不如授人以渔，本文主要讲解关于Linux系统下如何配置v2ray工具，假定你是一个拥有Linux基础的用户，且动手能力强，生命在于折腾。
- 本文采用docker容器安装方式，至于为何如此，我想你懂的。
<!--more -->
## 工具准备
1. 购买VPS或ECS。
前段时间买了国外几个服务商的试用了一下，总所周知的几个服务商的都不能用，后来选了一个小众 的折腾了一下，虽然勉强能用，但毕竟横跨太平洋，远程操作时那叫一个卡。后来报着试一试的态度买了阿里云香港的VPS。当然使用前考虑到毕竟是国内服务商，想必一定很严格，查了一下网上各种中奖。因此觉得常规的方式定然行不通，于是便想到了用`docker`试一试又何尝不可，毕竟容器单独隔离。
只是运行v2ray工具，一台小鸡足矣，配置如下：
> 内存1G
> CPU 1核
> 磁盘25G
> 流量1T
> 带宽30Mbps
> 系统Centos 7

2. 拉取v2ray镜像。
由于采用docker安装，首先得安docker软件。
```bash
yum install docker  # 安装docker
systemctl enable docker  # 设置开机启动
systemctl start docker  # 启动服务
```
国内用户docker拉取镜像时通常会很慢，这时我们得修改仓库地址为国内地址。在这里我们使用的是香港的服务器，自然也就不存在慢的情况了。
``` bash
docker pull v2ray/official   # 拉取镜像
```

3. 创建配置文件。
在VPS系统`/etc/v2ray/`目录创建配置文件`config.json`，写入以下格式内容。
```json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "dns": {
	"ip":"8.8.8.8",
	"port":53,
	"isDNS":true
  },
  "stats": {},
  "inbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "alterId": ***,
            "id": "***"
          }
        ]
      },
      "port": ***,
      "streamSettings": {
        "tcpSettings": {},
        "network": "tcp",
        "security": "auto"
      },
      "tag": "in-0"
    }
  ],
  "outbounds": [
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "outboundTag": "blocked",
        "ip": [
          "geoip:private"
        ],
        "type": "field"
      }
    ]
  },
  "policy": {},
  "reverse": {},
  "transport": {}
}
```
*修改配置中星号内容替换为自己的配置*。
`inbounds.clients.id`:uuid格式字符串
`inbounds.clients.alterId`:任意数字编号
`inbounds.port`:当前配置项对应端口号，尽可能四位以上，避免与系统端口冲突。
> inbounds:配置中该节点为一个数组对象，也就是说可以有多个配置项，可以是同种协议的，也可以是不同协议的，切记多个配置端口号不能相同，不同协议的配置能容略有差异。
[协议列表](https://www.v2ray.com/chapter_02/02_protocols.html)

## 启动容器
启动方式有多种，可以直接是命令行执行，也可以借用docker-compose工具启动配置好的容器。
- 命令行直接启动，后台执行且开机自启
``` bash
docker run -d --name v2ray --restart always -v /etc/v2ray:/etc/v2ray -p 16666:8888 -p 16668:8889 -p 16669:8890 v2ray/official v2ray -config=/etc/v2ray/config.json
```
> -d:后台运行
> --name:容器名称，不能重复
> --restart:容器会随着服务的启动而启动
> -v:挂载目录，系统目录:容器中目录，将系统目录`/etc/v2ray`挂载到容器中的`/etc/v2ray`目录，这样容器就能直接访问`/etc/v2ray`下的配置文件，当需要修改配置时直接修改系统目录下配置即可
> -p:暴露端口，系统端口:容器端口
> v2ray/official:容器启动的镜像
> v2ray -config=/etc/v2ray/config.json:容器执行的命令

至此工具已经配置完成，请记得检查系统是否已打开相应端口。及检查服务商后台防火墙或安全组是否已允许开放相应端口号，没有就加一下，不然访问不通。
安装[客户端工具](https://www.v2ray.com/awesome/tools.html)，配置相应配置即可。
