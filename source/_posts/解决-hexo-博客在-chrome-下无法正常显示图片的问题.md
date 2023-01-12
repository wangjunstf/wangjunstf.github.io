---
title: 解决 hexo 博客在 chrome 下无法正常显示图片的问题
date: 2021-11-05 20:09:05
updated:
categories: [疑难杂症]
tags: [chrome,hexo,next主题]
---
<meta name="referrer" content="no-referrer"/>

## 出现的问题

最近发现一个奇怪的问题， hexo 发的博客图片 在 Google Chrome 下无法正常显示，但在 Safari 和 FireFox 下确可以正常显示。
<!-- more -->
在 Chrome 下显示如下： 

<img src="https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%885.15.53.png" alt="截屏2021-11-05 下午5.15.53" style="zoom:50%;" />

## 出现问题的原因

我使用的腾讯云图床，并且开启了自定义域名的 CDN加速，但它并不支持 https 访问。

![截屏2021-11-05 下午6.01.54](https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%886.01.54.png)



查看下博客的网页源码，它的图片链接如下图所示：

```html
<p><img src="http://qqimage.wangjunblogs.com/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%883.27.01.png" alt="截屏2021-11-05 下午3.27.01"></p>
```

可以看到，它的链接是 http 开头的。



打开开发者选项：

![截屏2021-11-05 下午5.35.58](https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%885.35.58.png)

可以看到图片无法正常加载，看来原因就在这了。它的请求链接是 https 开头的，不对呀，它应该是 http 开头的呀。经过一段时间的摸索，终于知道原因了。

> 自 Chrome83 版本开始，Chrome浏览器默认阻止混合内容`MixContent`的下载，加载等。既 HTTPS 网站 只能以 HTTPS 加载内容。这个问题导致很多未配置静态资源SSL证书的网站在加载图片等http内容时失败。

## 解决方法

一种方式是更换浏览器，很明显对于用户来说产生了很大的不便，还是需要从网站管理员的角度来解决问题。



最佳方法：

使用基于 HTTPS 的图床。 

可以使用 github 图床，它默认支持 https。

或者使用腾讯云对象存储的默认上传地址：

打开文件基本信息，使用下图所示地址上传。 

![截屏2021-11-05 下午6.08.53](https://wangjun-1257394474.cos.ap-beijing.myqcloud.com/uPic/%E6%88%AA%E5%B1%8F2021-11-05%20%E4%B8%8B%E5%8D%886.08.53.png)

