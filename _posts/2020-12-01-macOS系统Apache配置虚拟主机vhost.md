---
layout: post
title:  "macOS系统Apache配置虚拟主机vhost"
date:   2020-12-01 16:55
categories: 不作恶 网站搭建

---

使用Apache来配置虚拟主机，基于IP、主机名或端口号，可以做到在单一服务器上运行多个网站。

1. 基于IP地址：一台服务器拥有多个IP地址，当用户访问不同IP地址时显示不同的网站页面。
2. 基于名称：Apache服务程序自动识别来源主机名或域名然后跳转到指定的网站。
3. 基于端口号：通过同一个IP不同端口号区分多个网站

本文介绍基于名称配置vhost，详细指导文档参见[官方文档](http://httpd.apache.org/docs/2.4/vhosts/)。

## 系统环境

> 操作系统：macOS Catalina 10.15.7
>
> web服务器： Apache 2.4
>
> web框架：ThinkPHP 6.0



## 配置Apache

### 更改httpd.conf

``` shell
# /etc/apache2/httpd.conf
# 1. 启用虚拟主机
LoadModule vhost_alias_module libexec/apache2/mod_vhost_alias.so
# 2. 打开重定向
LoadModule rewrite_module libexec/apache2/mod_rewrite.so
# 3. 从/etc/apache2/extral/httpd-vhosts.conf文件中导入配置
Include /private/etc/apache2/extra/httpd-vhosts.conf 
```

### 更改httpd-vhosts.conf

```shell
# /etc/apache2/extra/httpd.conf
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/docs/dummy-host.example.com"
    ServerName dummy-host.example.com
    ServerAlias www.dummy-host.example.com
    ErrorLog "/private/var/log/apache2/dummy-host.example.com-error_log"
    CustomLog "/private/var/log/apache2/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/docs/dummy-host2.example.com"
    ServerName dummy-host2.example.com
    ErrorLog "/private/var/log/apache2/dummy-host2.example.com-error_log"
    CustomLog "/private/var/log/apache2/dummy-host2.example.com-access_log" common
</VirtualHost>

# 新增配置
<VirtualHost *:80>
    ServerAdmin nineteenwj@gmail.com
    ServerName example.com
    ServerAlias 127.0.0.1
    # 网站路径
    DocumentRoot "/Users/My/Workspace/www/tp/public" 
    # 打开访问限制
    <Directory "/Users/My/Workspace/www/tp/public"> 
      Options -Indexes +FollowSymlinks
      AllowOverride All
      Require all granted
    </Directory>
    ErrorLog "/private/var/log/apache2/xnn.example.com-error_log"
    CustomLog "/private/var/log/apache2/xnn.example.com-access_log" common
    
    DirectoryIndex index.php
    # 可以在public路径下新增404.html等错误提示页面
    ErrorDocument 404 /404.html
    ErrorDocument 403 /403.html
</VirtualHost>
```

> - ServerAdmin: 管理员邮箱
> - ServiceName: 主机和端口名称，端口可省略
> - ServerAlias: ServiceName的别称，和ServiceName用途一致
> - DocumentRoot: 网站路径
> - ErrorLog: 用户日志文件（可选，如果不需要，则去掉该行）
> - CustomLog: 错误日志（可选，如果不需要，则去掉该行）

### 设置访问限制

httpd-vhosts.conf中将访问限制打开，但不能让用户无限制访问网站，因此需要使用.htaccess限制访问。

网站路径下创建.htaccess文件，见下

```shell
# /Users/My/Workspace/www/tp/public/.htaccess
<IfModule mod_rewrite.c>
  Options +FollowSymlinks -Multiviews
  RewriteEngine On

  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^(.*)$ index.php/$1 [QSA,PT,L]
</IfModule>
```

### 测试

配置完成后，重启apache，浏览器中输入 http://127.0.0.1，则将打开网站入口页面，如果访问的路径为非法，例如 http://127.0.0.1/home，则会跳转到404.html页面



