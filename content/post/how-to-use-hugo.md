+++
date = "2014-11-11T17:15:00+08:00"
draft = false
title = "使用Hugo搭建免费个人Blog"
tags = ["hugo", "golang"]
+++

# Hugo是什么

Hugo是一个工具，可以用于搭建静态站点，类似jekyll，不过Hugo是Golang写的，大家应该知道Golang有一个对部署友好的特点，那就是静态编译，所以安装起来非常方便，不像jekyll安装起来比较麻烦。

可能有些读者也不知道jekyll是干啥的，我这简单解释一下，这些软件通常可以叫做静态站点生成器，我们可以使用Markdown格式编写一些文本，按照指定的目录结构存放，然后再在指定的目录里放置css等静态文件，jekyll就可以帮你生成一个静态站点。那既然是静态站点，你就可以很方便的部署了，因为只要搭配一个web server即可，甚至可以部署在github pages上，[ops.m114.org](http://ops.m114.org)是部署在gitcafe pages上的，这样国人访问速度快一些。因为github和gitcafe的pages功能是免费的，这也是我标题中“免费”二次的原因

# Hugo的使用

Hugo的官网是[gohugo.io](http://gohugo.io/)，里边有个Docs，大家可以跟着走一遍，主要是里边的quickstart。笔者就不给大家做翻译了，给读者介绍一下如何基于笔者这个Blog来搭建，在这基础上修改就要容易不少了。

## 下载Hugo

官网上首页就有下载链接，去[Hugo下载](https://github.com/spf13/hugo/releases)即可

## 把笔者的这个blog clone下来

    git clone https://github.com/vinsonzou/hugo.blog.git
    cd hugo.blog
    hugo server -w

上面的代码是Linux、OS X控制台命令，windows用户请自己转换成windows操作方法。看到控制台打印出的内容了么？Hugo已经帮忙生成了一个静态站点，并且监听在本机的1313端口，访问一下试试吧：）

## 修改hugo.blog

一行代码就跑起来了，是不是so easy，接下来笔者大体介绍一下各个目录中的作用，读者可以修改成自己的一些信息

content目录就是存放你原始markdown文本的地方，content的子目录和markdown文件名组成了url地址，比如这篇文章的url是：http://ops.m114.org/post/how-to-use-hugo/ ，那是因为content目录下有个post/how-to-use-hugo.md

public目录是刚才运行`hugo server -w`命令生成的，这里边的内容就是静态站点的内容，之后咱们把这些内容提交到gitcafe pages中

static目录是存放一些静态资源

themes目录是主题目录，我使用了hyde这个主题，在上面做了一些修改，读者要想让Blog比较个性化，就可以定制主题

themes/hyde/{layouts,static}是我们主要修改的内容。index.html是首页，你修改一下看看，浏览器会自动刷新看到效果；partials目录是存放的一些页面片段，便于复用；_default目录是博文单页和博文列表页面，相信你一看就懂；static目录中有一些css，想怎么个性化就调整它们就成了

# 使用gitcafe pages制作站点

上面搞定之后，最好把修改之后的内容push到github上。public目录无需push，这是每次都可以自动生成的。咱们这里要把public也作为一个repo，push到gitcafe，生成静态站点。

gitcafe有个帮助文档：[GitCafe Pages](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9#wiki)，照着搞一下，把public的内容push上去，绑定域名，O了

是不是很简单，有明白的地方可以查看Hugo文档或留言。




