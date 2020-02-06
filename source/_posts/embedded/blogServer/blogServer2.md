---
title: 搭建自己的嵌入式博客服务器（二）环境搭建
categories:
  - 嵌入式
  - 博客服务
tags:
  - Hexo
  - 嵌入式
  - 博客服务
date: 2020-01-31 05:15:17
---
本篇简单介绍几个环境的搭建，包括node，Apache, hexo
<!-- more -->
## node.js和npm
这里使用的是Ubantu操作系统，其他Linux系统同理， (npm是node.js的包管理工具，随同node.js一同下载)
 1. 安装curl.
```powershell
sudo apt-get install curl 
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
```
 2. 安装nodejs和npm

```powershell
sudo    apt-get    install    nodejs-legacy
sudo    apt-get    install    npm
```

（1）安装用于安装nodejs的模块n

```powershell
sudo    npm    install    -g    n
```

（2）通过n模块安装指定的nodejs

```powershell
sudo    n    latest
sudo    n    stable
sudo    n    lts
```

（3）升级npm为最新版本

```powershell
sudo    npm    install    npm@latest    -g
```

 3. 确认

最后通过node -v ,npm -v确认版本，nodejs版本要在6.3以上才可以

接下来在bin下面创建个软连接

```powershell
/usr/local/bin/
ln -s /opt/node-v8.11.1-linux-x64/bin/hexo
```

## Apache2
 1. 下载

```powershell
sudo apt-get update
sudo apt-get install apache2
```

 2. 环境配置
 （1）/etc/apache2/apache2.conf 是主要配置文件(这个文件的末尾可以看到，include了其它所有的配置文件)。
（2）/etc/apache2/ports.conf 始终包含在主配置文件中。它用于确定传入连接的侦听端口，默认为80，我们一般都会重新配置新的端口。
（3）apache2的默认web目录：/var/www/html。（在/etc/apache2/sites-enabled/000-default.conf 里可以看到这个 DocumentRoot /var/www/html 配置）
（4）设置默认主页的配置文件/etc/apache2/mods-enabled/dir.conf
（5）进入到/etc/apache2/sites-available目录下，添加ErrorDocument 404 /404.html，在这里，你的404.html要在DocumentRoot 即可实现自定义404页面
按照自己的需求，配置上面的几个文件即可
 3. 开关

```powershell
sudo /etc/init.d/apache2 [ start | stop | restart | status ]
service apache2 [ start | stop | restart | status ]
```

 4. 验证
 在浏览器输入IP：端口号验证HTTP服务是否正常
 正常则会显示Ubantu系统默认http服务网页

## Hexo

 1. 安装
 
```powershell
sudo npm install hexo-cli -g
```
 2. 配置并初始化
 

```powershell
hexo init blog
cd blog
npm install
hexo server
```
执行完上面几条命令之后，在浏览器打开地址：http://localhost:4000/就会看到hexo为你提供的默认主题，则表示安装成功。
到 [hexo官网](https://hexo.io/themes/) 找个适合自己的主题。下载主题源码到  博客目录/themes/
具体的如何更改hexo主题，生成页面和文章可以参考[官方文档](https://hexo.io/docs/writing.html)

