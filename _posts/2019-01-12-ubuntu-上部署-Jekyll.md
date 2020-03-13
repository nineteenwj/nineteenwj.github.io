---
layout: post
title:  "ubuntu 上部署 Jekyll"
date:   2019-01-12 17:20
categories: jekyll 不作恶
---

ubuntu上部署jekyll非常简单，执行下列安装命令即可：

> sudo apt-get install ruby
>
> sudo apt-get install ruby-dev
>
> sudo apt-get install gems
>
> sudo gem install jekyll
>
> sudo gem install bundler
>
> sudo gem install minima

安装完毕后，做一个小小的测试，在当前路径新建一个blog，进入该路径，启动服务

> jekyll new blogtest
>
> cd blogtest
>
> jekyll serve

![jekyllserve]({{ site.url }}/assets/posts/ubuntu上部署Jekyll/jekyllserve.PNG)

打开浏览器，访问路径 http://127.0.0.1:4000 即可看到示例blog

![sampleblog]({{ site.url }}/assets/posts/ubuntu上部署Jekyll/sampleblog.PNG)

