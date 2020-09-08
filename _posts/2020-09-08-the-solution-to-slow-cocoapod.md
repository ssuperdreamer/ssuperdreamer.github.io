---
layout: post
title: "The solution to slow cocoapod"
subtitle: ""
date: 2020-09-08
author: "Axag"
header-img: "img/post-ios-solution.jpg"
tags: [iOS百宝箱]
---
### 问题起源

手头有个项目已经很久没有更新过第三方库了，基于最近Apple要求去除UIWebView的支持以及项目维护的需求，必须要升级第三方库，可是在升级的过程发现400M的更新内容却只有2~5k的更新速度，这不得不让我怀疑今年是不是2020年了，因此得想办法解决这个问题

### 原因

最核心的原因就是墙的问题,Cocoapod更新一直都比较慢，基于众所周知的原因，经常更新失败，特别cocoapod-1.8.0之后的版本还将原来从git下载repo master的方式调整为cdn，正常来说应该是大大的加快了pod setup,install,update的速度但是实际却发现`CDN: trunk URL couldn't be downloaded`这样的问题，解决了cdn的问题(添加host)却又出现了`https://cdn.jsdelivr.net/cocoa/Specs/a/7/5/AFNetworking/3.2.0/AFNetworking.podspec.json Response: SSL peer certificate or SSH remote key was not OK` 这样的问题，根据目前的情况我个人觉得下面这个解决方案应该是目前的最优解.

### 解决方案:翻墙 并且指定下载源（前提必须有良好的翻墙工具）

先移除cdn的源码

```shell
pod repo remove trunk
```

然后在pod file中添加数据源

```shell
source 'https://github.com/CocoaPods/Specs.git'
```

最后翻墙的工具有很多，每个人可以自行解决，翻墙后我们可以打开 `网络偏好设置=>高级=>代理=>查看下网页代理/安全网页代理`

![](/img/2020/20200531_3.png)

可以看到目前我的翻墙的本地代理是10080端口，然后我们在命令行将git 的网络代理设置为这个

```shell
git config --global http.proxy 127.0.0.1:10080
git config --global htts.proxy 127.0.0.1:10080
```

设置完后我们可以通过命令

```shell
cd ~
cat .gitconfig
```

查看下设置后是否成功

![20200531_1](/img/2020/20200531_1.png)

然后再执行pod update命令 我们会发现速度有了质的飞跃(这个和翻墙工具的速度有关)

![20200531_2](/img/2020/20200531_2.jpeg)

