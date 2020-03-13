---
layout: post
title:  "Windows上部署Jekyll"
date:   2019-01-22 23:30
categories: jekyll 不作恶
---

Jekylly 是一个简便易用免费的博客生成工具，并且可以部署到github上。但Jekyll对linux系统支持较大，windows上的版本更新不够及时，本篇文章即是记录如何在windows上安装Jekyll，供各位看官参考。 

## 1. 相关资源

Jekyll 官方中文地址：[https://www.jekyll.com.cn](https://www.jekyll.com.cn)

Jekyll windows安装指南：[http://www.madhur.co.in/blog/2011/09/01/runningjekyllwindows.html](http://www.madhur.co.in/blog/2011/09/01/runningjekyllwindows.html)

## 2. 安装步骤

windows上需要安装以下四个工具：

- 安装Ruby & Ruby Development Kit:[ https://rubyinstaller.org/downloads](https://rubyinstaller.org/downloads)  [Ruby 2.2.6 (x64)](https://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.2.6-x64.exe) 和[DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe](https://dl.bintray.com/oneclick/rubyinstaller/DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe) 此DevKit支持Ruby2.0 至2.3的版本，Ruby版本不能太高。
- 安装Jekyll
- 安装python（我使用的是python2.7.15）
- 安装Pygments

### 2.1 安装Ruby&DevKit

按以上链接下载Ruby和其DevKit安装包，双击安装Ruby

Ruby安装完成后，安装DevKit，将其解压到目录C:\rubydevkit下

进入C:\rubydevkit目录，执行命令 

> ruby dk.rb init

![rubyinit]({{ site.url }}/assets/posts/windows上部署Jekyll/rubyinit.JPG)

更改生成的config.yml文件，如下

```
# Example:
#
# ---
# - C:/ruby19trunk
# - C:/ruby192dev
#
---
- C:/Ruby22-x64
```

执行如下命令：

> c:\Ruby26-x64>ruby dk.rb init
> 
> c:\Ruby26-x64>ruby dk.rb review
> 
> c:\Ruby26-x64>ruby dk.rb install
> 
> c:\Ruby26-x64>gem -v
> 
> c:\Ruby26-x64>gem install jekyll

![rubyinstall]({{ site.url }}/assets/posts/windows上部署Jekyll/rubyinstall.JPG)

可能是自动安装的jekyll版本太高，依赖的ruby版本要求在2.3.0以上。网上查出ruby2.2.5与jekyll3.2.1匹配，尝试jekyll3.2.1版本，安装成功

> c:\Ruby26-x64>gem install jekyll -v '3.2.1'

![jekyll3.2.1]({{ site.url }}/assets/posts/windows上部署Jekyll/jekyll3.2.1.JPG)

在当前目录新建一个blog

> jekyll new blog

在当前路径下生成子目录myblog，进入该目录后执行启动服务

> jekyll serve

![bundlerError]({{ site.url }}/assets/posts/windows上部署Jekyll/bundlerError.JPG)

出错，需要安装bundler, bundler同样需要ruby的version >= 2.3.0，这里依然需要指定版本

> gem install bundler -v '1.11.0'

安装bundler后再试，仍然出错，需要安装minima

![minimaError]({{ site.url }}/assets/posts/windows上部署Jekyll/minimaError.JPG)

> gem install minima -v '1.2.0'

安装完成后启动jekyll服务

> jekyll serve

![jekyllserve]({{ site.url }}/assets/posts/windows上部署Jekyll/jekyllserve.JPG)

浏览器中打开URL http://127.0.0.1:4000/ 即可看到自动生成的默认Blog

![jekylldefaultblog]({{ site.url }}/assets/posts/windows上部署Jekyll/jekylldefaultblog.JPG)