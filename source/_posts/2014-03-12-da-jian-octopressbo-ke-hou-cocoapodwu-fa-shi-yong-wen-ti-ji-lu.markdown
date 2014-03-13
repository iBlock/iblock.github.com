---
layout: post
title: "搭建Octopress博客后cocoaPod无法使用问题记录"
date: 2014-03-12 22:48:51 +0800
comments: true
categories: [环境搭建, cocoapod, ruby]
---

今天在用pod安装第三方依赖库时发现pod无法使用了，出现类似下面错误：

{% blockquote %}
naxiannantekiMacBook-Pro:~ iBlock$ pod search
/Library/Ruby/Site/2.0.0/rubygems/dependency.rb:298:in to_specs': Could not find 'cocoapods' (>= 0) among 48 total gem(s) (Gem::LoadError)
from /Library/Ruby/Site/2.0.0/rubygems/dependency.rb:309:into_spec'
from /Library/Ruby/Site/2.0.0/rubygems/core_ext/kernel_gem.rb:53:in gem'
from /usr/bin/pod:22:in
{% endblockquote %}

看错误应该是ruby版本问题造成的，回想了下之前搭建octopress博客由于需要ruby1.9.3的环境，所以根据要求进行了安装，而cocopod需要的ruby是其它版本，估计是搭建Octopress博客时安装ruby1.9.3不知哪个步骤操作有误导致ruby2.0.0出现了问题，于是开始着手解决问题。

<!--more-->

首先输入下面命令查看ruby版本：

```
ruby --version
```
显示如下：

>ruby 1.9.3p545 (2014-02-24 revision 45159) [x86_64-darwin13.0.0]

看看当前系统上已安装的ruby版本列表：

{% codeblock %}
rvm list
{% endcodeblock %}

得到的结果如下：

{% blockquote %}
rvm rubies

   ruby-1.9.3-p545 [ x86_64 ]
   ruby-2.0.0-p451 [ x86_64 ]
=* ruby-2.1.1 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
{% endblockquote %}

发现ruby2.0.0版本是有的，切换ruby版本：

```
rvm use ruby 2.0.0
```

出错，错误码忘了保留了，总之提示ruby2.0.0版本有问题，输入以下命令重装下ruby2.0.0版本：

```
rvm reinstall ruby-2.0.0
```

该命令会将之前的ruby2.0.0版本删除重新下载安装。完成后切换到2.0.0版本，尝试使用pod search asiHttpRequest命令，发现还是有问题，差点就想重装下cocoapod，但是看到了cocoapod的升级命令，试试看能否升级修复下，如是输入：

```
sudo gem install cocoapods
```

命令完成后，再次输入pod search asiHttpRequest，成功了，输出如下内容：

{% blockquote %}
-> ASIHTTPRequest (1.8.1)
   Easy to use CFNetwork wrapper for HTTP requests, Objective-C, Mac OS X and
   iPhone.
   pod 'ASIHTTPRequest', '~> 1.8.1'
   - Homepage: http://allseeing-i.com/ASIHTTPRequest
   - Source:   https://github.com/pokeb/asi-http-request.git
   - Versions: 1.8.1 [master repo]
   - Sub specs:
     - ASIHTTPRequest/Core (1.8.1)
     - ASIHTTPRequest/ASIWebPageRequest (1.8.1)
     - ASIHTTPRequest/CloudFiles (1.8.1)
     - ASIHTTPRequest/S3 (1.8.1)
{% endblockquote %}

当想使用octopress的rake功能时只要使用如下命令切换ruby版本即可，OK，大功告成，记录一下。

```
rvm use 1.9.3
```