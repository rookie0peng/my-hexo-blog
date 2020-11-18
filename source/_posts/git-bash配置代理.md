---
title: git bash配置代理
date: 2020-11-19 03:32:28
tags: 
    - git
categories: 
    - program
---
### 1.https加速
##### 1-1.当代理为http或https时
1080为代理端口号。
```
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```
##### 1-2.当代理为ss或ssr时
注意，得确认自己的代理使用的是哪个协议，哪个端口，比如我的就是 socks5 和 1080，以及端口号。
```
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```
##### 1-3.重置代理
```
git config --global --unset http.proxy
git config --global --unset https.proxy
```
### 2.ssl加速
在用户文件夹下，打开/.ssh/config文件（如果没有就自己创建一个），输入以下内容。其中 id_rsa 文件需要换成你的文件，端口号 1080 也需要换成你的代理端口号。
```
Host github.com
User git
Port 443
Hostname ssh.github.com
IdentityFile ~\.ssh\id_rsa
TCPKeepAlive yes
ProxyCommand connect -S 127.0.0.1:1080 %h %p
```
如果使用Socks5代理，这时候测试可能会出现以下报错
```
FATAL: Cannot get password for user:xxx
```
原因是代理需要密码，需要设置一个环境变量SOCKS5_PASSWD。如果你的代理里没有设置用户名和密码的话随便填即可。
比如我的：
![图 2-1.png](https://blog-rookie0peng.oss-cn-shenzhen.aliyuncs.com/github/pages/2020/1119/27260163e0ca42359660bc6c768963ac.png)
