---
title: 搭建自己的嵌入式博客服务器（四）hexo安装及部署到GitHub
categories:
  - 嵌入式
  - 博客服务
tags:
  - Hexo
  - 嵌入式
  - 博客服务
date: 2020-02-06 07:07:54
---

到了博客搭建的最后一个阶段，使用hexo框架搭建完全静态博客站点,这里还简单介绍下将博客资源部署到github上的方法。
<!-- more -->
# 一，hexo的安装使用

1.安装Hexo

```powershell
npm install -g hexo-cli
```

之后使用hexo -v查看一下版本，版本显示成功则安装成功。

2.使用
首先要初始化整个模板

```powershell
hexo init myblog
```

myblog是你的的博客文件夹的名称，根据自己的喜好设置 

更新npm

```powershell
npm install
```

这里简单介绍下初始化好的博客文件夹下的几个目录

 - node_modules: 相关的依赖不常用 public：存放生成的页面
 - scaffolds：生成文章的一些模板，比如使用hexo    new "xx",xx.md
的默认内容可以通过修改模板来改变 

 - source：用来存放你的文章就是 xx.md文件
 - themes：存放主题

常用命令接介绍

```powershell
hexo clean  //清楚原有的静态缓存
hexo generate   //重新生成所有的静态网页
hexo deploy  //部署到相关的平台 例如 github
hexo new "xx"   //生成xx.md,新建文章
```

# 二，部署到GitHub（可以跳过，部署之后方便备份）

## 1，安装github（linux）

```powershell
sudo apt-get install git
```

 用git --version 来查看一下版本,显示版本代表安装成功。

## 2， 部署博客到GitHub

1.新建仓库
登录GitHub.com，点击New repository，新建仓库
创建一个和你用户名相同的仓库，仓库名称为 “用户名+.github.io”，这样你的博客才能通过GitHub page访问，GitHub page是GitHub新建项目后为项目建立的私人网站站点，通过 用户名+.github.io 访问时，会显示主目录下的index.html

之后create repository，创建完成。

2.生成SSH添加到GitHub

生成ssh key

```powershell
git config --global user.name "用户名"
git config --global user.email "邮件地址"
```

可以用以下两条，检查一下你有没有输对
```powershell
git config user.name
git config user.email
```

确认输入无误后，然后创建SSH

```powershell
ssh-keygen -t rsa -C "youremail"
```

一路回车，过程中会提示输入一些东西，使用默认配置即可，直接回车。

生成的ssh目录下回有id_rsa私钥和id_rsa.pub公钥，需要把这个公钥放在GitHub上，这样当你链接GitHub自己的账户时，公钥私钥匹配成功才能通过git上传你的文件到GitHub上。

在setting中，找到SSH keys的设置选项，点击New SSH key，把你的id_rsa.pub里面的信息复制进去。

```powershell
ssh -T git@github.com
```

提示（You've successfully authenticated, but GitHub does not provide shell access）,则表示测试成功。


3.将hexo部署到GitHub

打开hexo的根目录配置文件 _config.yml，添加以下内容

```powershell
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```


要想提交资源到github还需要安装deploy-git ，也就是部署的命令,这样你才能用命令部署到GitHub。

```powershell
npm install --save hexo-deployer-git
```

之后就可以使用hexo deploy /hexo d提交了

第一次提时，可能要你输入username和password。

提交成功后可以通过github查看是否提交成功，确认成功后可以通过用户名+.github.io查看你的博客了(注意这里是https)
