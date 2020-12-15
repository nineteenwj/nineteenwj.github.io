---
layout: post
title:  "macOS搭建Apache2.4+PHP7.3+MySQL8.0+ThinkPHP6.0开发环境"
date:   2020-12-02 16:33
categories: 不作恶 网站搭建


---

之前在windows上搭建过 apache+php+mysql 环境，换成macbook后一切都有所不同，配置方面有些许变化。本文只是做简单的搭建教程，不做深入探讨。

windows上如何搭建参见[《Win10下部署Apache+PHP+MySQL环境》](https://nineteenwj.github.io/archivers/Win10%E4%B8%8B%E9%83%A8%E7%BD%B2Apache+PHP+MySQL%E7%8E%AF%E5%A2%83)。

# 环境要求

## 开发环境

> 操作系统： macOS Cotalina 10.15.7
>
> web服务器： Apache 2.4
>
> 脚本： php 7.3.11 (cli)
>
> 数据库： MySQL 8.0.16
>
> web框架：ThinkPHP 6.0

## 工具安装

### Apache和PHP

使用的操作系统是macOS Catalina 10.15.7，自带Apache 2.4 和PHP 7.3.11

**测试Apache运行**

Terminal中执行指令

```shell
sudo apachectl start
```

浏览器中打开地址

```
http://localhost/
```

此时浏览器中显示以下内容，则表明Apache运行正常。

> It Works!

Apache 常用命令见下：

```shell
# 启动Apache服务  
sudo apachectl start
# 重启Apache服务  
sudo apachectl restart
# 停止Apache服务  
sudo apachectl stop
# 查看Apache服务  
sudo apachectl -v
```

### MySQL 8.0

可以选择直接到官网下载安装包，或是用brew 安装，可先用brew 查看哪些版本可用

```shell
antiwey@antiweys-MacBook-Pro WebServer % brew search mysql
==> Formulae
automysqlbackup            mysql-client@5.7           mysql@5.6
mysql                      mysql-connector-c++        mysql@5.7
mysql++                    mysql-sandbox              mysqltuner
mysql-client               mysql-search-replace
```

列表中并没有Mysql 8.0.16，直接在[官网](https://dev.mysql.com/downloads/mysql/)下载dmg安装文件，点击安装。

### ThinkPHP 6.0

**Composer下载及安装**

ThinkPHP 6.0的安装及使用需要用到composer，参考ThinkPHP [官方指导文档](https://www.kancloud.cn/manual/thinkphp6_0/1037481)。

Composer的官方下载命令见下：

```shell
curl -sS https://getcomposer.org/installer | php
```

由于国内下载速度太慢，可以使用国内镜像，请参见[官方国内镜像指导文档](https://pkg.phpcomposer.com/#how-to-use-packagist-mirror)进行安装，在此不赘述。

composer自动安装到了当前目录下，为使之全局可用，需要将其移动到环境变量PATH包含的路径下

```shell
sudo mv composer.phar /usr/local/bin/composer
```

如果网络条件好，可使用指令更新composer

```shell
composer selfupdate
```

或者在[官方国内镜像](https://www.phpcomposer.com/)下载并覆盖系统中的composer程序。

**ThinkPHP 6.0 安装**

执行以下命令:

```shell
composer create-project topthink/think tp
```

*tp*目录名可以任意更改，这个目录就是我们后面会经常提到的thinkphp应用根目录。

详细的安装教程参考[官方文档](https://www.kancloud.cn/manual/thinkphp6_0/1037481)

**测试ThinkPHP的运行**

在thinkphp的根目录中执行指令

```shell
php think run
```

浏览器中输入地址

```
http://localhost:8000/
```

正常情况下，会显示以下内容

> :)
>
> ThinkPHP V6.0.5
> 14载初心不改 - 你值得信赖的PHP框架
>
> [ V6.0 版本由 [亿速云](https://www.yisu.com/) 独家赞助发布 ]
>
> [官方教程系列](https://e.topthink.com/api/go/ed141c5b8eca350ce)[官方应用市场](https://e.topthink.com/api/go/ec6544b97d9c94a40)[统一API调用服务](https://e.topthink.com/api/go/e7e1354fa534b4eaa)

# 服务配置

Apache可以直接跑ThinkPHP，但不够灵活，可将Apache配置成虚拟主机，用以配合ThinkPHP 6.0的多App方案，从而达到一个服务器管理多个网站的目的。

> Apache直接启动ThinkPHP的见文章最末“Apache直接启动ThinkPHP”。

## Apache配置虚拟主机

使用Apache来配置虚拟主机参见[《macOS系统Apache配置虚拟主机vhost》](https://nineteenwj.github.io/archivers/macOS%E7%B3%BB%E7%BB%9FApache%E9%85%8D%E7%BD%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BAvhost)。

## php配置

**更改配置文件**

```shell
# /etc/apache2/httpd.conf
# 打开加载php7模块,该库存放路径在 /usr/libexec/apache2
LoadModule php7_module libexec/apache2/libphp7.so
```

**pdo_mysql**

php使用macOS Catalina自带的php7.3.11(cli)。各种教程里都提到需要打开php.ini中的extension=pdo_mysql，但macOS里无需做这一步，如果打开，反而会报如下错误：

```shell
PHP Warning:  PHP Startup: Unable to load dynamic library 'pdo_mysql' (tried: /usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql (dlopen(/usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql, 0x0009): dlopen(): file not found: /usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql), /usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql.so (dlopen(/usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql.so, 0x0009): dlopen(): file not found: /usr/lib/php/extensions/no-debug-non-zts-20180731/pdo_mysql.so)) in Unknown on line 0
```

应该是根本用不上pdo_mysql库。

## MySQL 配置

**更改连接认证方式**

如果直接使用php链接mysql，会报如下错误：

```php
PHP Warning:  mysqli_connect(): The server requested authentication method unknown to the client [caching_sha2_password] 
```

Mysql 8.0 系统默认的验证方式从mysql_native_password 换成了caching_sha2_password，但php 7.3并不支持这样的验证方式，需要将用户的验证方式改成mysql_native_password。

查看当前mysql用户的认证方式：

```mysql
mysql> select user,host,plugin from mysql.user;
+------------------+-----------+-----------------------+
| user             | host      | plugin                |
+------------------+-----------+-----------------------+
| test             | %         | caching_sha2_password |
| mysql.infoschema | localhost | caching_sha2_password |
| mysql.session    | localhost | caching_sha2_password |
| mysql.sys        | localhost | caching_sha2_password |
| root             | localhost | caching_sha2_password |
+------------------+-----------+-----------------------+
```

更改已有账户test的认证方式：

```mysql
mysql> ALTER USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY 'new password';
mysql> select host,user,plugin from mysql.user;
+------------------+-----------+-----------------------+
| user             | host      | plugin                |
+------------------+-----------+-----------------------+
| test             | %         | mysql_native_password |
| mysql.infoschema | localhost | caching_sha2_password |
| mysql.session    | localhost | caching_sha2_password |
| mysql.sys        | localhost | caching_sha2_password |
| root             | localhost | caching_sha2_password |
+------------------+-----------+-----------------------+
```

更改后，即可使用php连接mysql 8.0

> NOTE：如果在开发或者配置过程中报找不到mysql头文件，可以将mysql安装目录中的头文件软链接到系统路径中。
>
> ln -s /usr/local/mysql/include/* /usr/local/include/

**配置ThinkPHP 数据库设置**

Thinkphp的环境配置文件在根目录下，将.example.env 改为.env，并匹配数据库信息

```shell
# tp/.env
APP_DEBUG = true
[APP]
DEFAULT_TIMEZONE = Asia/Shanghai
[DATABASE]
TYPE = mysql
HOSTNAME = 127.0.0.1
DATABASE = servername
USERNAME = test
PASSWORD = test
HOSTPORT = 3306
CHARSET = utf8
DEBUG = true
[LANG]
default_lang = zh-cn
```



配置完成后，重起apache服务，在服务器中输入打开http://127.0.0.1即可打开thinkphp默认页面。

> :)
>
> ThinkPHP V6.0.5
> 14载初心不改 - 你值得信赖的PHP框架
>
> [ V6.0 版本由 [亿速云](https://www.yisu.com/) 独家赞助发布 ]
>
> [官方教程系列](https://e.topthink.com/api/go/ed141c5b8eca350ce)[官方应用市场](https://e.topthink.com/api/go/ec6544b97d9c94a40)[统一API调用服务](https://e.topthink.com/api/go/e7e1354fa534b4eaa)



# Apache 直接启动 ThinkPHP

这里介绍一种简单的使用Apache启动ThinkPHP的方式。

**更改apahce配置文件**

```shell
# /etc/apache2/httpd.conf
# 配置根目录，也就是刚刚composer下载的thinkphp public路径，public中存放着thinkphp的入口文件 index.php
DocumentRoot "/Users/antiwey/Workspace/Develop/www/tp/public"
<Directory "/Users/antiwey/Workspace/Develop/www/tp/public">
```

**重启Apache**

```shell
sudo apachectl restart
```

**打开页面**

​      浏览器地址栏输入 http://localhost，得到和之前测试thinkphp一样的内容

> :)
>
> ThinkPHP V6.0.5
> 14载初心不改 - 你值得信赖的PHP框架
>
> [ V6.0 版本由 [亿速云](https://www.yisu.com/) 独家赞助发布 ]
>
> [官方教程系列](https://e.topthink.com/api/go/ed141c5b8eca350ce)[官方应用市场](https://e.topthink.com/api/go/ec6544b97d9c94a40)[统一API调用服务](https://e.topthink.com/api/go/e7e1354fa534b4eaa)

# 注意事项

1. **tp/runtime** 目录需要设置为可写权限，否则tp框架运行会出错。（包括log写入及session的使用）

2. 如需使用session，需打开middleware.php的session开关

   ```php
   # /tp/app/middleware.php
   <?php
   // 全局中间件定义文件
   return [
       // 全局请求缓存
       // \think\middleware\CheckRequestCache::class,
       // 多语言加载
       // \think\middleware\LoadLangPack::class,
       // Session初始化
       \think\middleware\SessionInit::class
   ];
   ```

   

   