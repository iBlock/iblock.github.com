---
layout: post
title: "IOS Provisioning Profiles描述文件到期解决方法"
date: 2014-04-03 23:40:19 +0800
comments: true
categories: [环境搭建,苹果证书相关]

---

从上周开始，公司陆续有人跟我说发布在公司网站上的IOS应用程序无法下载安装，提示172.16.xx.xxx证书无效，起初我没在意，后来又有人跟我反应说安全应用客户端在iPhone上打开闪退，前两天还能正常使用。这接二连三的异常现象让我不得不开始重视起来，自从公司的安全客户端某个版本发布后一直就没动过了，怎么最近开始出现这种异常问题，而且是有的说是下载不了，有的说是在iphone上打开就闪退，我开始排查问题，到今天为止把问题都解决了，做个记录。本篇文章主要解决客户端在iPhone上闪退的问题。

<!--more-->

经过问题排查最后发现闪退现象是因为Provisioning Profiles描述文件到期了，之前打包时使用的Provisioning Profiles文件于2013年3月27日到期，所以3月27日之后用户打开应用由于文件到期导致应用直接闪退。解决这个问题很简单，首先登录苹果开发者网站，找到Provisioning Profiles，因为是打包文件，所以选择Distribution，如下图所示：

![alt text](/images/2014/04/04/Provisioning Profiles.png "Provisioning Profiles")

找到之前的描述文件，这个时候只要将之前的描述文件重新生成一下即可，不用重新新建一个描述文件。重新生成之后描述文件的日期将会延后一年，然后将这个描述文件下载到本地，导入到你要打包的机器上，然后重新打一个包，让之前应用打开会闪退的iPhone手机重新安装新打的包即可。仅仅是为了记录一下，避免以后再次出现这种问题又花费时间排查。

至于文章开头说的另一个错误：公司网站上的IOS应用程序无法下载安装，则可参考我的另一篇博客[IOS7.1 企业应用无法安装解决方案](http://iblock.github.io/blog/2014/04/04/ios7-dot-1-qi-ye-ying-yong-wu-fa-an-zhuang-jie-jue-fang-an/)


