---
layout: post
title: "Octopress使用过程中出现的问题"
date: 2014-10-08 22:34:04 +0800
comments: true
categories: [环境搭建, Octopress]
keywords: Octopress
 
---

好久没写博客了，想要记录一下最近学习的一些东西，熟练的进到octopress目录下敲入rake preview，直接就报错了，根据错误提示重新bundle update一下就好了。再次敲入rake preview命令，命令没有报错，然后我在safari中输入 http://127.0.0.1:4000/ 进行查看，又出现一个麻烦事，整个页面显示空白，不知什么原因，只能请教google大神了，好在查找的第一条记录就找到问题的解决方案了，找到octopress目录下的Gemfile文件，在最后加上 

	gem 'thin'

保存文件，在命令行输入bundle update，再次打开地址发现一切正常了，可以开始写博客了。