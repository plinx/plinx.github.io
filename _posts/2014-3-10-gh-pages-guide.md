---
layout: post
title: Github Pages Blog
---

本文主要讲述如何制作 `Github Pages Blog`。

过程主要有三大部分：

- 创建Github Pages
- 套用Jekyll/Jekyll-bootstrap
- 使用域名


##1、创建Github Pages

Q:[Github Pages](http://pages.github.com/) 是什么？

A:Github Pages 是 github 推出的一款页面托管服务，可以作为项目介绍，也可以用于制作 blog。

git 可谓是程序员项目管理必不可少的工具之一，如果你还没用过git，那可能需要[恶补一下](恶补一下 "http://baike.baidu.com/subview/1531489/12032478.htm?fr=aladdin")了。

关于 git 与 github 的用法不再多提， 要进入下一步， 首先你需要一个[github账号](https://github.com/)。

以下步骤仅讨论 Win7 环境，你也可以参考[官方文档](http://pages.github.com/)。

下载一个 [git for windows](http://msysgit.github.io/)，之后以 `$开头` 的都是 `Git Bash` 下的命令行操作。

在 github 上创建一个新的 repository :

![create repo](/img/gh-pages guide/create repo.jpg)

repository 需命名为 `username`.github.io。(username 需要更改为你自己的 github 用户名)

接着在本地操作 Git Bash :

{% highlight bash %}
$ ssh-keygen -t rsa -C "email_name@your_email.com"
{% endhighlight %}

有提示全部回车就可以了，接着会生成 SSH key.
	
{% highlight bash %}
$ cd ~/.ssh
$ ls
id_rsa	id_rsa.pub ...
$ pwd
/c/Users/name/.ssh
{% endhighlight %}

`id_rsa.pub` 就是我们需要的文件了，而 /c/Users/name/.ssh/ 就是 id_rsa.pub 的存放目录。

打开id_rsa.pub（可以用记事本，或者其他文本编辑器），将内容复制。

登陆github，进入 Acount Setting， Add SSH Key，复制内容即为 Key。

![login](/img/gh-pages guide/acount setting.jpg)
![sshk](/img/gh-pages guide/ssh keys.jpg)

![pssh](/img/gh-pages guide/ssh keys add.jpg)

测试一下本地 git 与 github 是否成功连同 :

{% highlight bash %}
$ ssh -T git@github.com
{% endhighlight %}

配置本地 git ：

{% highlight bash %}
$ git config --global user.name "your_name"
$ git config --global user.email "email_name@your_email.com" 
{% endhighlight %}

接着就可以开始创建你的主页了 :

{% highlight bash %}
$ git clone https://github.com/username/username.github.io
$ cd username.github.io
$ git remote add orgin master

$ echo "hello github pages" > index.html
$ git add -am "hello pages"
$ git push origin master
{% endhighlight %}

首次创建，同步需要10分钟，过了10分钟之后，登陆 `username`.github.io 就可以看到你的主页了。

##2、套用Jekyll/Jekyll-bootstrap

Q:为什么是 Jekyll？
A:Jekyll 是 github 官方支持的，而且 Jekyll 本身还支持 markdown，用起来非常方便，你也可以看到我完成状态下的写作环境(Git Bash + Markdown + Browser)如下图。

![wimg](/img/gh-pages guide/wimg.jpg)


详细的步骤你也可以参考 Jekyll [官方文档](http://jekyllrb.com/docs/quickstart/) 与 Jekyll-bootstrap [官方文档](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)。

官方文档是直接从 Jekyll 应用开始的，在此之前我们还需要安装两样东西 ：

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [Ruby Gems](http://rubygems.org/pages/download)

Tips:

{% highlight bash %}
安装 Ruby 的时候需要勾选 Add ... PATH 与 Associate .rb ...
{% endhighlight %}

安装完所需要的东西之后，你可以试一试官方的 [Quick Start](http://jekyllrb.com/docs/quickstart/) :
	
{% highlight bash %}
$ gem install jekyll
$ jekyll new testblog
$ cd testblog
$ jekyll serve
{% endhighlight %}

接着我们可以在浏览器中查看 http://localhost:4000 页面了。

尝试过之后我们 clone 一份官方模板。

{% highlight bash %}
$ git clone https://github.com/plusjade/jekyll-bootstrap.git
$ cd jekyll-bootstrap
$ jekyll serve
{% endhighlight %}

可以看到官方模板的页面了。

如果你需要查看更多的官方主题，可以根据[官方说明](http://jekyllbootstrap.com/usage/jekyll-theming.html)来使用。

我用的模板是官方主题 `mark-reid`，不过自己对主题的一些元素(背景，文字)进行了定制，这只需要一点点 css 的基础。

在定制主题的过程中，主要需要修改以下几个文件：

{% highlight bash %}
_include/themes/mark-reid/default.html
assert/themes/mark-reid/css/screen.css
{% endhighlight %}

还需要修改主页的可以更改 index.md 文件。

- 当我们需要写博客的时候，只需要在 _post 目录下，加入命名为 Year-Mon-Day-Title.md 的 markdown 文档就行了就行了。

例如我在写的这篇 ：

{% highlight sh %}
2014-3-9-gh-pages-guide.md
{% endhighlight %}

Jekyll-bootstrap 还有很多用法，如果想要做一个优秀的个人博客，需要更加深入的学习，限于时间，我只能将最简单的用法讲清楚。

Jekyll 的一个使用技巧 ：
{% highlight sh %}
$ jekyll serve --watch &
{% endhighlight %}

加上 `--watch` 可以使 localhost:4000 页面刷新即变化，不需要重新执行 serve。

在整个安装过程中，不同电脑肯能还会遇上不同的坑，这些自行搜索解决。

在这里，我提三个大家可能遇到的 ：

- 中文页面无法显示

解决方法 ： 编辑 `_config.yml`，加入一句
```
enconding: utf-8
```

参考 [_config.yml configuration](http://jekyllrb.com/docs/configuration/)

- 'cannot load such file -- wdm (LoadError)'

解决方法 ： gem 安装

{% highlight bash %}
$ gem install wdm
{% endhighlight %}

- git push 时，返回邮件 The submodule `_theme_packages/mark-reid` was not properly initialized with a `.gitmodules` file.

原因 ： 我们使用的是 bootstrap 的模板，在使用过程中，我们还通过 rake 来安装了主题，这些主题都是属于不同的 github 树的，我们可以在主题目录中使用 `$ ls -a`来查看，会发现存在 `.git` 目录。
解决方法 ： 清除子模块 或 将子模块加入主 git module

{% highlight bash %}
$ rm -rf _theme_packages/mark-reid/.git
Question : 这样是否涉及协议问题，我在文档中的 README 未看到相关信息。
{% endhighlight %}

或者

{% highlight bash %}
$ cd _theme_packages/mark-reid/
$ git submodule init
{% endhighlight %}




##3、使用域名
**这一步并不是必要的，我们的页面已经挂在 username.github.io 上面了，完全可以把这个就当做你的 Blog 页。**

当然，如果你想要个性化，花几十块置办个域名也不会太贵。

闲话少说，进入正题。

域名购买，我查阅的很多博客都推荐 Goddady(域名服务商) + Dnspod(DNS服务商) 来解决，大家可以尝试。

我是第一次购买域名，所以直接在万网买了，万网是阿里云旗下的，支付什么的也很方便，以下的设置我以万网为例。

购买了域名后，我们首先需要设置解析。

有两种解析方法：

- A记录解析

使用A记录解析，我们需要指向 IP，指向的 IP 为 Github 的服务 IP。

这个 IP 经常变，所以大家需要查阅 Github Pages Help 来确定，[直通车](https://help.github.com/articles/my-custom-domain-isn-t-working)。

A记录解析，就是我们申请域名的一级域名解析，即 example.com。

所以我们需要在 username.github.io 根目录下加入一个文件 `CNAME`，加入一句 :

{% highlight html %}
example.com
{% endhighlight %}

- CNAME解析

使用CNAME解析，我们需要指向一个域名，这个域名就是我们的 username.github.io。

所以我们根目录下的 `CNAME` 应该加入的是 :

{% highlight html %}
www.example.com
{% endhighlight %}

我采用的是CNAME解析，最终如下图 :

![cname](/img/gh-pages guide/cname.jpg)

接着就可以等待我们的域名生效了。

整个过程就是这么简单，你还可以参考以下的一些优秀博客 :

1、[使用Github Pages建独立博客](http://beiyuu.com/github-pages/)

2、[zipperary's blog](http://zipperary.com/categories/hexo/)

3、[阮一峰的网络日记](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)

4、[hzmook](http://hzmook.github.io/2012/07/01/use-jekyll-build-blog-on-github.html)


原文地址 ： [www.cplinx.com/2014/03/10/gh-pages-guide](/2014/03/10/gh-pages-guide)

&copy; 2014 plinx
