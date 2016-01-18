---
layout: post
title: "React-Native环境搭建"
date: 2016-01-18 21:40:05 +0800
comments: true
categories: [React-Native, hybrid, 环境搭建]

---

![alt text](/images/2016/01/18/ReactNative.png "ReactNative")

##前言
最近领导让我们做一个H5与Native交互的方案，他最初的要求其实很简单，只要能在H5调起我们终端的支付功能进行支付就行。我的想法就是基于webView做一个URL拦截处理，与服务端协商好事件类型，如1是支付，2是弹窗，对URL中的类型与参数进行处理即可，当然这种做法比较low，不过可以很快完成需求。然后基于这个需求我搜了下目前网上的一些方案，发现了本文的主题，由facebook开源的React-Native框架，这个框架允许你使用 JavaScript 开发原生的 iOS 应用，同时也支持Android。

<!--more-->

##安装步骤
* ###安装Node版本管理器-> NVM

####1、首先进入.git目录下，该目录在/User/当前用户目录下，需要打开隐藏文件权限。如果没有该目录，则先创建该目录
{% codeblock %}
git init
{% endcodeblock %}
####2、从git仓库中clone工程到本地
{% codeblock %}
git clone https://github.com/cnpm/nvm.git
{% endcodeblock %}
####3、在终端中输入命令nvm，这时会发现提示如下
{% codeblock %}
-bash: nvm: command not found
{% endcodeblock %}
####4、上述的错误是提示nvm命令找不到，可以通过以下命令指定nvm路径
{% codeblock %}
source ~/.nvm/nvm.sh
{% endcodeblock %}
这时重新在终端输入nvm，会显示一些nvm的命令帮助，证明nvm已经能找到并使用。但是使用这种方式仅在本次生效，也就是说当把终端关闭再次打开时，再输入nvm又会发现命令找不到了。为避免每次都要指明nvm.sh的位置可以为命令终端添加启动配置。打开终端输入以下命令
{% codeblock %}
open -e !$
{% endcodeblock %}
将会打开profile文件，添加
{% codeblock %}
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
{% endcodeblock %}
* ###安装Node

####1、在终端输入以下命令等待执行完成即可完成Node安装
{% codeblock %}
nvm install node && nvm alias default node
{% endcodeblock %}
####2、查看下Node是否生效，输入node -v，如果能显示版本号则正常

* ###使用 homebrew 安装 watchman，一个来自Facebook 的观察程序以及类型检查程序：

{% codeblock %}
brew install watchman
brew install flow
{% endcodeblock %}
通过配置 watchman，React 实现了在代码发生变化时，完成相关的重建的功能。就像在使用 Xcode 时，每次保存文件都会进行一次创建。

* ###安装 React-Native

通过 npm安装即可：
{% codeblock %}
npm install -g react-native-cli
{% endcodeblock %}

* ###初始化一个项目

{% codeblock %}
react-native init AwesomeProject
{% endcodeblock %}


