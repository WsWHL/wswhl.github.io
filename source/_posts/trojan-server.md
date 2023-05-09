---
title: Trojan服务部署
date: '2020-04-05 14:46:56'
updated: '2020-04-05 14:46:56'
categories: 
  - 科学上网
tags:
  - Trojan
  - Proxy
---
# trojan-gfw

[trojan-gfw](https://github.com/trojan-gfw/trojan)
<!--more -->
- config

	``` json
	{
		"run_type": "server",
		"local_addr": "0.0.0.0",
		"local_port": 443,
		"remote_addr": "trojan-nginx",
		"remote_port": 80,
		"password": [
			"mypassword"
		],
		"log_level": 1,
		"ssl": {
			"cert": "/cers/tls.crt",
			"key": "/cers/tls.key",
			"key_password": "",
			"cipher": "ECDHE-ECDSA-AES128-GCM-SHA256****",
			"cipher_tls13": "TLS_AES_128_GCM_SHA256***",
			"prefer_server_cipher": true,
			"alpn": [
				"http/1.1"
			],
			"reuse_session": true,
			"session_ticket": false,
			"session_timeout": 600,
			"plain_http_response": "",
			"curves": "",
			"dhparam": ""
		},
		"tcp": {
			"prefer_ipv4": false,
			"no_delay": true,
			"keep_alive": true,
			"reuse_port": false,
			"fast_open": false,
			"fast_open_qlen": 20
		},
		"mysql": {
			"enabled": false,
			"server_addr": "trojan_db",
			"server_port": 3306,
			"database": "trojan",
			"username": "trojan",
			"password": "123456"
		}
	}
```
** 将tls证书放置`/cers/`目录*

# nginx

- 安装nginx软件包
`yum install nginx -y`

- 启动nginx服务

	``` shell
	systemctl enable nginx
	systemctl start nginx
```

# trojan service

创建服务配置文件
`vim /etc/systemd/system/trojan.service`

- 服务配置

	``` shell
	[Unit]
	Description=trojan
	Documentation=man:trojan(1) https://trojan-gfw.github.io/trojan/config https://trojan-gfw.github.io/trojan/
	After=network.target

	[Service]
	Type=simple
	StandardError=journal
	User=nobody
	AmbientCapabilities=CAP_NET_BIND_SERVICE
	ExecStart=/usr/bin/trojan -c /etc/trojan/config.json
	ExecReload=/bin/kill -HUP $MAINPID
	Restart=on-failure
	RestartSec=3s

	[Install]
	WantedBy=multi-user.target
	```

- 服务启动命令

	``` shell
	systemctl enable trojan
	systemctl start trojan
	```
