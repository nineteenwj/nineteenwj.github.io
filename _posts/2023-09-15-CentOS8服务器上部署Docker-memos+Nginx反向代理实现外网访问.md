---
layout: post
title:  "CentOS8服务器上部署Docker memos+Nginx反向代理实现外网访问"
date:   2023-09-15 14:23
categories: 不作恶 网站搭建

---

## 什么是memos

如果你用过flomo，则对memos不会陌生，它的功能跟flomo不能说一模一样，只能说基本相同。

简单来说，memos是一个开源免费的flomo，当然如果你没用过flomo，也可以直接把memos看做是一个私人微博，也跟微博一样，有三种模式private模式(仅自己可见)、group模式（visible to members）、public模式（完全公开）。默认是private模式。

交互界面是这样的，非常简洁。

![memos页面](/assets/posts/CentOS8服务器上部署Dockermemos+Nginx反向代理实现外网访问/memos页面.png)

memos主页：[https://usememos.com/](https://usememos.com/)

Memos github 地址: [https://github.com/usememos/memos](https://github.com/usememos/memos)

想要部署memos，有很多种方式：

1. 自己搭 
2. Docker 官方推荐的方式
3. Replit 参见仓库 [https://github.com/SinzMise/memos-on-replit](https://github.com/SinzMise/memos-on-replit)

如果想要加入memos的开发者行列，需要掌握以下技术：

**memo的技术栈** 

|Frontend                       | Backend                        |
|------------------------------ | ------------------------------ |
| [React](https://react.dev/)         | [Go](https://go.dev/)             |
| [Tailwind CSS](https://tailwindcss.com/) | [SQLite](https://www.sqlite.org/) |
| [Vite](https://vitejs.dev/)          |                                   |
| [pnpm](https://pnpm.io/)                 |                                   |

**知识储备**

- [Go](https://golang.org/doc/install)
- [Air](https://github.com/cosmtrek/air#installation) for backend live reload
- [Node.js](https://nodejs.org/)
- [pnpm](https://pnpm.io/installation)

本人不懂React、TailwindCSS、Vite、go语言等，最快的方式只能是老老实实按照官方的部署教程部署。



## Docker安装

参见 [CentOS8 安装 Docker-阿里云开发者社区](https://developer.aliyun.com/article/753261)

可能会出错，参见[Cent OS 8安装docker并解决docker和podman冲突问题_packages providing this file are: 'podman-docker' _tianshuiyim](https://blog.csdn.net/tianshuiyimo/article/details/116846613)

```JSON
Problem 1: problem with installed package podman-2:4.2.0-1.module_el8.7.0+1216+b022c01d.x86_64   - package podman-2:4.2.0-1.module_el8.7.0+1216+b022c01d.x86_64 requires runc >= 1.0.0-57, but none of the providers can be installed
...
```

## 运行Memo的Docker容器

**Docker run 方式运行**

要使用Docker运行部署memos，最简单的方式是运行以下命令：

```JSON
docker run -d \
  --init \
  --name memos \
  --publish 5231:5230 \
  --volume ~/.memos:/var/opt/memos \
  ghcr.io/usememos/memos:latest
```

这将在后台启动memos，并运行在宿主机（服务器）Host的5231端口上。数据将存储在`~/.memos`目录中。可以根据需要更改端口和数据目录的路径。仅需更改第一个端口，例如`5231:5230`，Host机中的端口5321，第二个端口5320是memos在容器内部监听的端口。对于目录也是如此，第一个路径`~/.memos`是Host系统上的路径，第二个路径`/var/opt/memos`是容器内部的路径。

成功运行后，可以通过[http://IP:Port](http://IP:Port) 进行访问，或是绑定该IP的域名**www.your_domain.com**进行访问，如果Host侧的端口不是80，则需要在域名后添加端口号，如映射关系是`--publish 5231:5230`，则访问地址为：[http://www.your_domain.com:5231](http://www.your_domain.com:5231)，如果端口号直接映射成80 `--publish 80:5230`，则可直接通过IP或是域名进行访问，注意，此时是非HTTPS访问。

NOTE: 如果需要更改容器的参数，比如映射端口号，并使改动生效，则需要先删除`docker rm <容器ID或容器名称>`，再重新创建新容器。

## **使用Nginx反向代理memos**

memos应用运行后，可以使用Nginx创建一个反向代理，将一个域名连接到memos实例上。

> 如何在CentOS8上安装nginx，见[安装nginx](#安装nginx)

宿主机Host和Docker之间的关系见下简单示意图：

![host与docker的关系图](/assets/posts/CentOS8服务器上部署Dockermemos+Nginx反向代理实现外网访问/host与docker的关系图.png)

nginx会将端口5231的内容转换为80和443端口，供外部访问。

1\. **创建配置文件**

创建一个名为`/etc/nginx/sites-available/www.your_domain.com`的文件，`www.your_domain.com`文件内容见下：

```JSON
server {
    listen 80;  # 监听 80 端口，这一行可写可不写，默认监听80端口
    server_name your_domain.com;

    location / {
        proxy_pass http://localhost:5231;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

创建新路径（如果没有的话）`/etc/nginx/sites-enabled/` ，并创建软连接

``` 
mkdir /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/www.your_domain.com /etc/nginx/sites-enabled/www.your_domain.com
```

2\. **修改nginx.conf**

在`/etc/nginx/nginx.conf`中添加`include /etc/nginx/sites-enabled/*;`，删除或注释掉原有listen 80端口的配置：

```JSON
http {
  ...
  include /etc/nginx/sites-enabled/*;
  # server {
  #     listen       80 default_server;
  #     ...
  #}
}
```

完成后，运行`sudo systemctl start nginx`开启nginx服务，通过地址 [http://www.your_domain.com](http://www.your_domain.com/) 或[http://服务器IP]()即可访问memos。

### 配置SSL安全证书

若要使用SSL安全证书，简单的方式是从你的域名服务商网站上直接下载，腾讯云或阿里云上都可，我使用的是腾讯的域名，因此在腾讯云上下载。

ssl安全证书的文件有4个，只用到其中2个：

```JSON
your_domain.com.csr
your_domain.com.key   <--- 要用这个
your_domain.com_bundle.crt   <--- 要用这个
your_domain.com_bundle.pem
```

通过winscp或其他方式将`your_domain.com_bundle.crt` 和 `your_domain.com.key` 上传到服务器的路径下`/etc/nginx/ssl/`中。

更改`/etc/nginx/sites-available/www.your_domain.com`内容，添加ssl配置。

```Python
server {
    listen 443 ssl;
    server_name your_domain.com;

    #请填写证书文件的相对路径或绝对路径
    ssl_certificate /etc/nginx/ssl/your_domain.com_bundle.crt;
    #请填写私钥文件的相对路径或绝对路径
    ssl_certificate_key /etc/nginx/ssl/your_domain.com.key;
    ssl_session_timeout 5m;
    #请按照以下协议配置
    ssl_protocols TLSv1.2 TLSv1.3;
    #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://localhost:5231;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

修改完成后运行`nginx -s reload`重载nginx配置，然后通过 [https://www.your_domain.com](https://www.your_domain.com) 即可安全访问。

### Http 重定向到 https中

用户可能会尝试使用http访问网站，但https是更安全的访问方式，因此需要将http重定向到https。

更改`/etc/nginx/sites-available/www.your_domain.com`内容，添加http重定向配置。

```JSON
server {
    listen 443 ssl;
    listen 80;
    server_name your_domain.com;

    #请填写证书文件的相对路径或绝对路径
    ssl_certificate /etc/nginx/ssl/your_domain.com_bundle.crt;
    #请填写私钥文件的相对路径或绝对路径
    ssl_certificate_key /etc/nginx/ssl/your_domain.com.key;
    ssl_session_timeout 5m;
    #请按照以下协议配置
    ssl_protocols TLSv1.2 TLSv1.3;
    #请按照以下套件配置，配置加密套件，写法遵循 openssl 标准。
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    
    location / {
        proxy_pass http://localhost:5231;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 配置HTTP请求跳转到HTTPS
    if ($scheme = http) {
        return 301 https://$host$request_uri;
    }
}
```

运行`nginx -s reload`重载nginx配置。

## 升级memos到最新版本

要升级memos到最新版本，需要首先停止并删除旧的memos容器：

``` bash
docker stop memos && docker rm memos
```

建议在升级前备份数据库：

```bash
cp -r ~/.memos/memos_prod.db ~/.memos/memos_prod.db.bak
```

然后拉取最新的镜像：

```bash
docker pull ghcr.io/usememos/memos:latest
```

再记得将数据库还原。

最后，按照之前的`docker run`命令重新启动memos。

## 其他补充

### 安装nginx

1\. 打开终端，以超级用户或具有sudo权限的用户身份登录。

2\. 首先，更新系统的包管理器缓存，以确保你获取到最新的软件包信息：

```Bash
sudo dnf update
```

3\. 安装 Nginx：

```Bash
sudo dnf install nginx
```

在安装过程中，系统会提示你确认安装。输入 `y` 并按 Enter 键继续。

4\. 安装完成后，启动 Nginx 服务并设置它在系统启动时自动启动：

```Bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

这将启动 Nginx 并确保它会在系统启动时自动运行。

注意：此时不能开启其他网络服务，比如apache。

5\. 检查 Nginx 是否正在运行：

```Bash
sudo systemctl status nginx
```

如果 Nginx 正在运行，你将看到一条消息显示为 "active (running)"。

6\. 配置防火墙规则以允许HTTP流量（如果防火墙启用）：

```Bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

这将确保防火墙允许HTTP请求通过。

7\. 在浏览器中访问服务器的IP地址或域名，以确认 Nginx 是否已成功安装并运行。默认情况下，Nginx 的默认网页根目录位于 `/usr/share/nginx/html`。

### 停止Docker

如何停止运行docker？

1\. 停止一个容器，其中 `<容器ID或容器名称>` 应该替换为你要停止的容器的实际容器ID或容器名称：

```Bash
docker stop <容器ID或容器名称>
```

例如，如果你的容器名称是 "memos"，则可以运行：

```Bash
docker stop memos
```

如果你不确定容器名称或ID，可以使用 `docker ps` 命令列出正在运行的容器，并找到你要停止的容器的信息。

2\. Docker 将发送停止信号给容器，容器会停止运行。你可以使用以下命令来验证容器是否已停止：

```Bash
docker ps -a
```

如果你的容器不再显示在列表中，表示它已成功停止。

请注意，使用 `docker stop` 命令会优雅地停止容器，让容器有机会完成正在进行的任务和清理资源。如果需要立即强制停止容器，可以使用 `docker kill` 命令。

```Bash
docker kill <容器ID或容器名称>
```

但要小心使用 `docker kill`，因为它会立即终止容器，可能导致数据丢失或不完整的关闭操作。通常，首选使用 `docker stop` 来正常停止容器。

### 进入Docker

进入运行中的 Docker 容器，可以使用 `docker exec` 命令。以下是进入 Docker 容器的一般步骤：

1\. 首先，打开终端窗口。
2\. 查看正在运行的容器。

```JSON
docker ps
CONTAINER ID   IMAGE                           COMMAND     CREATED         STATUS         PORTS                                       NAMES
fa0ebb35202b   ghcr.io/usememos/memos:latest   "./memos"   2 minutes ago   Up 2 minutes   0.0.0.0:5230->5230/tcp, :::5230->5230/tcp   memos
```

3\. 使用以下命令进入已运行的 Docker 容器：

```Bash
docker exec -it <容器ID或容器名称> /bin/sh
```

在上面的命令中，`-it` 标志允许你交互式地进入容器，`<容器ID或容器名称>` 应该替换为你要进入的容器的实际容器ID或容器名称。

例如，上面查到容器名为`memos`：

```Bash
docker exec -it memos /bin/sh
```

4\. 当你运行上述命令时，你将进入容器的命令行界面，可以在其中执行容器内的命令。

5\. 当你完成容器内的任务后，可以键入 `exit` 命令来退出容器，回到主机的命令行界面。

请注意，要使用 `docker exec` 进入容器，容器必须是运行状态。如果容器已停止，你需要首先使用 `docker start` 命令来启动它，然后再使用 `docker exec` 进入容器。

另外，要进入容器，你可能需要具有足够的权限，具体取决于容器内部的用户和权限设置。一些容器可能需要使用 `sudo` 命令或以 root 用户身份进入。

### 查看当前Docker容器的信息

要查看当前容器的系统信息，你可以在**容器内部**运行一些系统命令来获取相关信息。以下是一些常见的系统信息查询命令以及如何在容器中运行它们：

1\. 查看操作系统版本：

```Bash
cat /etc/os-release
```

这个命令将显示容器中的操作系统版本信息。

2\. 查看内核版本：

```Bash
uname -a
```

这个命令将显示容器内核的版本和其他系统信息。

3\. 查看CPU信息：

```Bash
cat /proc/cpuinfo
```

这个命令将显示有关容器中 CPU 的详细信息，包括型号、核心数等。

4\. 查看内存信息：

```Bash
cat /proc/meminfo
```

这个命令将显示容器内存的详细信息，如总内存、可用内存等。

5\. 查看磁盘空间：

```Bash
df -h
```

这个命令将显示容器中的磁盘空间使用情况。

6\. 查看网络信息：

```Bash
ifconfig
```

这个命令将显示容器的网络接口信息。