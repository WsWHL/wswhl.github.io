title: ClasshX协议配置
---
### VLESS

```yaml
  - name: <名称>
    type: vless
    server: <服务地址>
    port: <服务端口>
    uuid: <标识ID>
    alterId: 64
    cipher: auto
    udp: true
    network: ws
    sni: <服务地址>
    tls: true
    ws-opts:
      path: /<回退路径>
      headers:
        Host: <服务地址>
```