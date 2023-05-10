title: 解决Mac默认vim无法使用剪贴板问题
date: '2020-04-05 14:50:49'
updated: '2020-04-05 14:50:49'
tags:
  - Mac OS
  - Vim
categories:
  - 每日一记
---
## 开篇

mac系统自带的vim属于阉割版，不支持一些功能，因此需要手动编译替换。
查看默认vim版本及已启用功能：
``` shell
vim --version
```
*输出功能前面带符号`+`表示已启用该功能。例如：输出包含`+clipboard`表示支持剪贴板复制功能，否则不支持。*
<!--more -->
## 源码编译

1. 下载源码
源码地址：https://github.com/vim/vim/releases
``` shell
curl -LJO https://github.com/vim/vim/archive/v8.2.0352.tar.gz	#下载
tar -zxvf vim-8.2.0352.tar.gz	#解压
```

2. 设置编译参数
需要支持的功能就在此处设置
``` shell
./configure --with-features=huge --enable-pythoninterp=yes  --enable-cscope --enable-fontset --enable-perlinterp --enable-rubyinterp --with-python-config-dir=/usr/lib/python2.7/config --prefix=/usr/local
```
> 默认已启用剪贴板复制功能，所以不用设置

3. 编译和安装
``` shell
make && make install
```

4. 替换默认vim
``` shell
echo "alias vim='/usr/local/bin/vim'" >> ~/.zshrc #默认文件为.bash_profile
source ~/.zshrc
```
