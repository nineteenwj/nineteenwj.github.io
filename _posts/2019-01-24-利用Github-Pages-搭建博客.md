---
layout: post
title:  "利用 Github Pages 搭建博客"
date:   2019-01-24 22:35
categories: github-pages 不作恶
---
听说github pages还不错，稍微做些了解，自由度高，但是不如网页博客那样方便编辑，但是支持markdown。只要前期搭建好后，后期新增内容应该很方便吧。废话不多说，先来试试吧

### 创建Github账号

github 官网在此：[https://github.com](https://github.com) ，具体创建流程不做赘述

### 创建简易github page

官方教程在此：[https://pages.github.com](https://pages.github.com)

第1步创建仓库时，**Repository name** 中的填写格式为 *username.github.io* ，*username*是github上的用户名，这也是博客的访问地址。

第2步克隆仓库时，可以采用命令行形式或者下载github 桌面软件，我这里采用的TortoiseGit，也比较方便。其中涉及到ssh key的设置，在这里不做赘述。

在仓库根目录下创建index.html，将如下代码写入index.html中

```html
<!DOCTYPE html>
<html>
<body>
<h1>Hello World</h1>
<p>I'm hosted with GitHub Pages.</p>
</body>
</html>
```

将index.html push到github的仓库上后，访问地址 [https://username.github.io](https://username.github.io) 即可看到如下页面：

![helloworldpage]({{ site.url }}/assets/posts/利用GithubPages搭建博客/helloworldpage.JPG)




### 搭建网页静态框架

实际上如果熟悉前端开发，已经可以在github page的仓库中搭建自己的静态网页，但如果是作为博客网页，每篇文章都需要自己手动排版并生成html太过麻烦，毕竟html的网页排版并不算友好。因此可以引入github推荐的免费博客生成工具jekyll，辅助制作博客页面，支持markdown，文章书写将会变得方便很多。

免费版Github中，有很多功能（比如wiki，部署jelly）都无法使用，因此只能自己手动部署jekyll。没有金钱辅助，只能时间凑。

Jekyll的安装教程见下方。




### 相关文章
[ubuntu上部署Jekyll](https://nineteenwj.github.io/archivers/ubuntu-%E4%B8%8A%E9%83%A8%E7%BD%B2-Jekyll)



[windows上部署Jekyll](https://nineteenwj.github.io/archivers/windows-%E4%B8%8A%E9%83%A8%E7%BD%B2-Jekyll)


