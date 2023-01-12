---
title: ssh配置免密登录
date: 2021-07-14 21:50:44
updated:
categories: [Linux]
tags: [Linux,ssh,免密登录]
---
通过配置密钥，可以免密登录远程主机。

在需要登录远程主机的终端输入命令：
<!-- more -->
## 第一步：生成本机密钥
```
ssh-keygen -t rsa   #rsa指定加密类型
```

会有几个选项，回车即可
看到该图案，代表配置成功：

```
+---[RSA 3072]----+
|        o=+      |
|         o.     .|
|        .  .  o +|
|       . ... . +=|
|        S +..o=++|
|         o =++.=*|
|          * o..*+|
|         + + .=o=|
|        .Eoo*oo=+|
+----[SHA256]-----+
```

![](https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/ssh-keygen.png)

进入密钥所在目录:

## 第二步：复制公钥

```
.
├── config
├── id_rsa
├── id_rsa.pub
├── known_hosts
└── known_hosts.old
```

id_rsa为私钥，只能自己所有，不能分享给别人
id_rsa.pub为公钥，部署到其它主机就可以与本机安全通信。

公钥和私钥的区别：**私钥加密，公钥解密**

> 1. **Bob将他的公开密钥传送给Alice**。
> 2. **Alice用Bob的公开密钥加密她的消息，然后传送给
>    Bob**。
> 3. **Bob用他的私人密钥解密Alice的消息**。

打开id_rsa.pub，复制其内容

## 第三步：服务器配置

执行和本机一样的命令，并按回车：

```
ssh-keygen -t rsa   #rsa指定加密类型
```

生成了.ssh文件，进入该目录

```
cd ~/.ssh
vim authorized_keys
```

将第二步复制的公钥粘贴进authorized_keys，注意不要粘贴多余的字符，比如空格或者回车。

