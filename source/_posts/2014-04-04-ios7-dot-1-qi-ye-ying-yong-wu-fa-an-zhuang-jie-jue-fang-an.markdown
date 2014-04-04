---
layout: post
title: "IOS7.1 企业应用无法安装解决方案"
date: 2014-04-04 15:34:33 +0800
comments: true
categories: [环境搭建, 苹果证书相关]

---

本文章为转载，主要参考["这里"](http://blog.csdn.net/zhaoxy_thu/article/details/21133399)。

最近有人反馈说公司网站上发布的IOS应用下载时提示证书错误，无法下载，于是我用我的手机试了下却发现可以正常下载，和有问题的手机比较排查了下发现只要是升级到了IOS7.1系统的iPhone手机都无法下载，之前的都是可以正常下载的，在网上搜了下解决方案，果然是因为这个，简单的说就是ios7.1要安装企业应用，url必须是https的，不能是http，这就要求服务器需要支持https。因此只要将原链接：
{% codeblock lang:text %}
itms-services://?action=download-manifest&url=http://example.com/manifest.plist
{% endcodeblock %}
改为：
{% codeblock lang:text %}
itms-services://?action=download-manifest&url=https://example.com/manifest.plist
{% endcodeblock %}
即可。

对于服务器，则需要增加对https的支持，下面我就将我搭建的步骤记录一下。

<!--more-->

###一、安装配有SSL模块的apache版本
本人使用的是[httpd-2.0.65-win32-x86-openssl-0.9.8y.msi](http://mirrors.hust.edu.cn/apache//httpd/binaries/win32/httpd-2.0.65-win32-x86-openssl-0.9.8y.msi)
点击上面链接下载安装，安装过程较为简单，在这里就不做说明了。

###二、修改apache的配置文件
1、找到apache安装目录下的conf/httpd.conf，打开文件，去掉以下内容前的#号
{% codeblock lang:text httpd.conf %}
LoadModule ssl_module modules/mod_ssl.so
{% endcodeblock %}

并在文件最后加上：

{% codeblock lang:text httpd.conf %}
<VirtualHost *:8080>
    ServerAdmin xxx@xxx.com（随意）
    DocumentRoot C:/Server（服务器根目录）
    ServerName 192.168.xxx.xxx（服务器域名或ip地址）
    ErrorLog logs/test-error_log
    CustomLog logs/test-access_log common
    SSLEngine on
    SSLCertificateFile "C:/Program Files/Apache Group/Apache2/conf/ssl.crt/server.crt"（之后生成证书的完整路径）
    SSLCertificateKeyFile "C:/Program Files/Apache Group/Apache2/conf/ssl.key/server.key" （之后生成密钥的完整路径）
</VirtualHost>
{% endcodeblock %}

2、修改conf/ssl.conf文件的以下内容：（以下为修改完的，可参考）

{% codeblock lang:text ssl.conf %}
#SSLSessionCache        none
#SSLSessionCache        shmht:logs/ssl_scache(512000)
SSLSessionCache        shmcb:logs/ssl_scache(512000)
#SSLSessionCache         dbm:logs/ssl_scache
...
SSLCertificateFile conf/ssl.crt/server.crt
...
SSLCertificateKeyFile conf/ssl.key/server.key
{% endcodeblock %}

3、在conf目录下创建ssl.crt和ssl.key目录（不创建也行，只要保证以上两个路径和之后的文件路径对应即可）

###三、在命令行下切换到apache目录下的bin目录，运行以下命令
1、生成服务器的私钥
{% codeblock lang:text %}
openssl genrsa -out server.key 1024
{% endcodeblock %}
以上命令会在bin目录下生成server.key文件

2、生成签署申请
{% codeblock lang:text %}
openssl req -new –out server.csr -key server.key -config ..\conf\openssl.cnf
{% endcodeblock %}
输入以上命令回车后会开始让你填入相关信息。（注意除Common Name以外可以为空，Common Name必须为服务器的ip或域名）执行完后会在当前目录下生成server.csr文件

3、生成私钥
{% codeblock lang:text %}
openssl genrsa  -out ca.key 1024
{% endcodeblock %}
上面命令会在当前目录下生成ca.key文件

4、利用CA的私钥产生CA的自签署证书
{% codeblock lang:text %}
openssl req  -new -x509 -days 365 -key ca.key -out ca.crt  -config ..\conf\openssl.cnf
{% endcodeblock %}
输入以上命令回车后会开始让你填入相关信息。（注意除Common Name以外可以为空，Common Name必须为服务器的ip或域名）执行完后会在当前目录下生成ca.crt文件

5、在当前目录创建demoCA，里面创建文件index.txt和serial，serial内容为01，index.txt为空，以及文件夹newcerts

6、CA为网站服务器签署证书
{% codeblock lang:text %}
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -config ..\conf\openssl.cnf
{% endcodeblock %}
上面的命令会在当前目录下生成server.crt文件

7、最后将server.crt和server.key复制到上文对应的路径下
{% codeblock lang:text %}
conf/ssl.crt/server.crt
conf/ssl.key/server.key
{% endcodeblock %}

8、重启Apache服务器，即增加了https的支持。可以在浏览器访问[https://localhost:8080](https://localhost:8080)试试。如果不行，可以在logs\test-error_log文件中看看出了什么错误。

###四、将CA证书安装到iPhone上
最后，我们要将自己创建的CA证书安装到iphone上。将上面生成的ca.crt文件通过以下方式安装到iPhone手机上：


1、邮件方式，用iPhone自带的Mail程序打开（别的邮件客户端不行）

2、将证书放到发布应用的网站上，用safari浏览器打开并安装证书

安装完证书后再次访问我们之前的itms-services链接，就可以正常安装了。