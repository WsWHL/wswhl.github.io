title: hexo博客在线编辑同步服务hexon部署
date: '2023-05-07 09:01:44'
updated: '2023-05-07 09:01:48'
tags:
  - Blog
  - hexon
  - hexo
  - linux
categories:
  - 每日一记
---
hexo框架是一个主题丰富, 内容简洁的轻量级静态博客框架. 但也正是如此在实际使用中缺陷也很明显, 那就是博客文章内容不方便管理, 每次编写好markdown文件还需要经过hexo命令编译打包推送等步骤, 虽然命令不多, 但体验很不好, 有种很明显的脱离感. 而hexon的出现很好的弥补了这些不足之处, 为[hexon](https://github.com/gethexon/hexon)作者点个大大的赞👍!!!
<!-- more -->

至此, 当我们部署hexon服务后我们可以实现如下效果:
- 同步博客git内容, 实现在线编辑并推送, 配置好GitHub Action后可以自动触发部署
- 通过hexon编辑博客文章后, 直接在我们的服务器进行打包部署, 编译好的内容将直接推送到GitHub仓库的`gp-pages`分支实现自动部署, 接管GitHub Action构建功能

✨准备内容:
- 💻一台公网vps服务器, 1G足矣
- 🙎GitHub账号, 并初始化好hexo博客仓库
- 👋Linux命令基础, 会动的小指头
- 🤔️一颗善于思考的小脑瓜...

项目依赖项`git`、`pnpm`、`node.js`、`hexo`

1. 安装`pnpm`包管理工具
   ```shell
   bash wget -qO- https://get.pnpm.io/install.sh | ENV="$HOME/.bashrc" SHELL="$(which bash)" bash -
   ```
   这里仅以linux 64位平台举例,具体平台参考官网: https://pnpm.io/installation

2. 拉取博客项目
   默认采用root用户,后续hexon服务拉取和提交博客代码均需要访问GitHub仓库,建议提前配置好服务器的ssh key访问配置, 确保git命令能正常操作仓库
   ```shell
   git clone git@github.com:<github_name>/<github_name>.github.io.git
   npm install -g hexo-cli      # 安装hexo工具
   npm install      # 还原博客项目依赖
   hexo server   # 可检查博客服务是否能正常启动
   ```
   项目同步git时将与拉取分支保持一致, 部署时需要在`_config.yml`配置中添加如下`gh-pages`配置信息
   ```text
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:<github_name>/<github_name>.github.io.git
  branch: gh-pages
   ```


1. 拉取hexon项目并还原依赖, 初始化博客配置
   ```shell
   git clone https://github.com/gethexon/hexon  # 拉取项目
   pnpm install     # 还原依赖项
   pnpm run setup       # 初始化博客项目路径等配置信息
   ```
2. 启动hexon服务
   ```shell
   pnpm start
   ```
   默认配置监听本地5777端口,浏览器访问: http://localhost:5777

#### 进阶教程, 采用systemd部署

1. 创建hexon服务配置
   创建文件`/etc/systemd/system/hexon.service`并写入如下配置
      ```text
   [Unit]
   Description=Hexon Service
   After=network.target

   [Service]
   Environment=NODE_ENV=production
   Environment=DEBUG=null
   Environment=PATH=/root/.local/share/pnpm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
   WorkingDirectory=/opt/hexon/server
   Type=simple
   User=root
   Group=root
   Restart=on-failure
   RestartSec=5s
   ExecStart=/root/.local/share/pnpm/node dist/index.js
   LimitNOFILE=1048576

   [Install]
   WantedBy=multi-user.target
   ```
   WorkingDirectory: 设置为我们拉取hexon项目时的文件夹, 启动命令会在该目录下执行
   User、Group: 服务执行用户信息, 尽量保持和我们在终端执行时的一致, 因为存在上下文依赖关系, 比如git拉取提交项目时需要ssh密钥授权等
   > 文件中PATH追加了pnpm环境路径`/root/.local/share/pnpm`很重要, 后续hexon需要执行该路径下的一些命令, 具体路径为我们安装pnpm时所使用的用户主目录下隐藏文件夹中
2. 启动hexon守护服务
   ```shell
   systemctl enable hexon     # 添加自启
   systemctl start hexon      # 启动服务
   ```
   > 启动后查看日志确认是否正常执行`journalctl -u hexon -f`

关于hexon不足之处的几点建议:
1. 希望能够支持插入图片, 并能够配置图床(阿里云、腾讯云等文件存储服务).
2. 希望能够支持hexo项目设置项的可视化配置, 以此更加方便管理和配置
3. 希望登录授权支持双因子认证(2FA), 从而提高后台服务安全性
4. 其他内容待后续充分体验后补充...

[鸣谢项目hexon!!!](https://github.com/gethexon/hexon)
