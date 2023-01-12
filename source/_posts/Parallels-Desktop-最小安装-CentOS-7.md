---
title: Parallels Desktop 最小安装 CentOS 7
date: 2021-11-05 17:36:45
updated:
categories: [Linux]
tags: [CentOS7]
---
<meta name="referrer" content="no-referrer"/>

## 镜像文件下载

下载地址：http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/

BT种子：http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.torrent
<!-- more -->
## 完整性确认

确认是否被篡改：

```shell
$ shasum -a 256 CentOS-7-x86_64-Minimal-2009.iso 
07b94e6b1a0b0260b94c83d6bb76b26bf7a310dc78d7a9c7432809fb9bc6194a  CentOS-7-x86_64-Minimal-2009.iso
```

sha256sum.txt

```
07b94e6b1a0b0260b94c83d6bb76b26bf7a310dc78d7a9c7432809fb9bc6194a  CentOS-7-x86_64-Minimal-2009.iso
```

两者相同，说明未被篡改。

## 安装

### 新建虚拟机

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8A%E5%8D%8811.45.34.png" alt="截屏2021-11-05 下午4.26.34" style="zoom:33%;" />



### 选择镜像文件

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8A%E5%8D%8811.46.08.png" alt="截屏2021-11-05 下午4.26.34" style="zoom:33%;" />

继续，选择快速安装

### 设置密码，安装位置默认

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%882.40.11.png" alt="截屏2021-11-05 下午4.26.34" style="zoom:33%;" />

继续

### 自动安装即可

输入账号密码即可进入系统



## 安装 parallels tools 

操作菜单，选择安装 parallels tools 



查看系统是否存在光驱

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%883.23.35.png" alt="截屏2021-11-05 下午3.23.35"  />



可见系统用 sr0 表示光驱，接下来挂载光驱，将其挂载到 /media 文件下。

> Linux 系统一切皆文件，包括设备。当我们要访问某个设备，就将它挂载到一个文件，然后通过这个文件来访问这个设备。

![截屏2021-11-05 下午3.27.01](https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%883.27.01.png)

提示：/dev/sr0 是写保护的，只读挂载，这里没有问题，因为我们只需读取它的安装文件。



接下来就可以安装 parallels tools  ，直接下一步开始安装。



出现以下页面表示，安装成功，直接回车即可。

<img src="https://raw.githubusercontent.com/wangjunstf/pics/main/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%883.38.16-20211105171504693.png" alt="截屏2021-11-05 下午4.26.34" style="zoom:50%;" />