---
layout: post
title: "Octopress博客开博"
date: 2014-03-10 22:06:21 +0800
comments: true
categories: 随笔杂谈
keywords: Octopress
description: Octopress博客搭建

---

![alt text](/images/20140310/octopress.png "octopress")

<!--more-->

##前言
octopress弄了好几天终于搭建起来了，主要是一开始在捣鼓各种各样的主题，然后找各种各样的插件，到头来发现我并不需要这些，我仅仅是需要一个能记录我的生活、记录我学习的点滴这么一个平台。为什么会选择Octopress？虽说只是为了写博客，但是我之前试过在其它的平台比如：博客园、CSDN等尝试写博客，但是效果很不理想，无论是外观样式还是排版都不怎么满意。我一直在看[唐巧](http://blog.devtang.com/ "唐巧")和[OneV's Den](http://onevcat.com/ "上善若水")写的技术博客，发现这样的博客正是我想要的样子，马上一通Google，最终才找到了octopress，下面我将把我搭建该博客的步骤记录下来。

##安装步骤
* ####安装RVM，RVM(Ruby Version Manager)负责安装和管理Ruby的环境
{% codeblock %}
curl -L https://get.rvm.io | bash -s stable --ruby
{% endcodeblock %}

* ####安装Ruby 1.9.3
{% codeblock %}
git clone git://github.com/imathis/octopress.git octopress 			#octopress路径可以根据需要修改
cd octopress 		#进入到指定路径下
ruby --version 		#应该能正确输出ruby版本号才对
{% endcodeblock %}

* ####安装相关依赖项
{% codeblock %}
gem install bundler
rbenv rehash
bundle install
{% endcodeblock %}

* ####安装默认的Octopress主题，其它主题可参考[这里](https://github.com/imathis/octopress/wiki/3rd-Party-Octopress-Themes, "主题列表")
{% codeblock %}
rake install
{% endcodeblock %}

##配置Octopress
我这里主要是针对默认主题来展开的，其它主题的配置可能不一定适用。

* ####首先修改_config.yml文件
我主要修改以下几项：
`title`、`subtitle`、`author`
，并且删除twitter相关的信息，由于GFW的原因，将会造成页面load很慢。

关于_config.yml文件中的更多内容，请看[这里](http://octopress.org/)

##将博客部署到github上
* ####准备工作

1.创建`github`账号

2.在github上`创建一个仓库`，并将仓库名称按照这样的方式进行命名：`username.github.com`或`organization.github.com`，之后我们的博客就可以以`http://username.github.com`这样的方式来进行访问了。
以上内容如有不明白的可自行Google。

* ####根据Octopress的`rake`命令来自动配置仓库
{% codeblock %}
rake setup_github_pages
{% endcodeblock %}
上面的命令主要做的就是创建一个_deploy目录，目录用来存放部署到master分支的内容。期间会要求你输入仓库的url，根据提示，进行输入即可。
完成上面的命令之后，我们就可以生成博客并真正的部署到仓库中了。执行如下命令：
{% codeblock %}
rake generate
rake deploy		#通过该命令把博客生成的内容提交到master分支下
{% endcodeblock %}
上面的命令首先生成博客文件，并将生成的博客文件拷贝到`_deploy/`目录下，然后将这些内容添加到git中，并commit和push到仓库的`master`分支。
现在可以访问`http://username.github.com`了。注意：有时候可能会有延时，要等几分钟才能打开（我等了10分钟左右）。

在使用rake deploy命令时可能会出现如下错误：
{% blockquote %}
Pushing generated _deploy website
To git@github.com:xxx/iblock.github.com.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:xxx/iblock.github.com.git'
{% endblockquote %}

看了很多人的博客搭建上都没有提到这个问题，不知道是否都没遇到，总之我是遇到了，出现这个问题后会发现怎么使用rake deploy这个命令在github的maste分支都没有任何东西，经过Google，最后找到解决方案了，修改`rakefile`文件里面的system "git push origin #{deploy_branch}"改成system "git push origin `+`#{deploy_branch}"就解决了。

* ####将博客的源代码放到source分支下
{% codeblock %}
$ git add .
$ git commit -m 'Initial source commit'
$ git push origin source
{% endcodeblock %}

如果在部署到仓库之前，需要先预览一下博客，可以在终端输入`rake preview`命令，然后就能在浏览器中进行本地预览访问了：`http://127.0.0.1:4000/`或`http://localhost:4000/`，效果跟仓库中的一样。

完成上面的步骤后已经可以开始写博客了，只不过是页面也太过精简了，下面是用octopress写博客的几个步骤：
{% codeblock %}
$ rake new_post["New Post"]
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source
$ rake deploy
{% endcodeblock %}

##一些个性化的定制
* ####添加最近更新
修改_config.yml文件，找到`default_asides:`，添加asides/recent_posts.html

* ####修改插件用来支持中文分类
修改octopress/plugins目录下的category_list.rb插件，修改如下：
{% codeblock lang:ruby category_list.rb %}
require 'stringex'

module Jekyll
  class CategoryListTag < Liquid::Tag
    def render(context)
      html = ""
      categories = context.registers[:site].categories.keys
      categories.sort.each do |category|
        posts_in_category = context.registers[:site].categories[category].size
        category_dir = context.registers[:site].config['category_dir']
        # category_url = File.join(category_dir, category.gsub(/_|\P{Word}/u, '-').gsub(/-{2,}/u, '-').downcase)
        category_url = File.join(category_dir, category.to_url)
        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
      end
      html
    end
  end
end

Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
{% endcodeblock %}

#困了....未完待续....