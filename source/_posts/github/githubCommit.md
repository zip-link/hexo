---
title: 菜鸟入门github上传本地项目
categories:
  - github
tags:
  - github 
  - 代码库
date: 2020-02-08 08:10:45
---
首先通过github管理你的项目的前提是你要有一个github账户并且你的电脑或者服务器已经安装了git
以下按提交资源到github的先后顺序介绍几个git命令
<!-- more -->
1.首先需要初始化git文件

```powershell
git init
```

执行成功后，你的项目目录下会生成一个.git的隐藏文件。

2.然后可以通过add命令添加新增文件到本地的github缓存

```powershell
git add .    //这里使用. 代表所有文件，当然也可以添加特定的单个或多个文件
```

3.对比本地资源的提交差异

```powershell
git status
```

该命令会把你本地工作区和暂存区的版本进行比较，查看当前的状态。我下面的状态是已经把所有文件加入到了暂存区中，但是还没有提交到本地历史区。

4.提交差异

```powershell
git commit -m "这里是注释。。。"
```

该命令会把本地暂存区中的文件提交到本地历史区，注意只有在本地历史区中的内容才能提交到github。执行该命令后，我们所有的文件都只是在本地。没有github任何关系。

5.提交本地差异到github的仓库暂存区

```powershell
git remote add github项目URL
```

6.先同步github文件

```powershell
git pull --rebase origin master 
```

如果对比commit的文件与服务器文件有差异，这条命令会先将差异打包，之后同步，最后将差异合并。

7.提交之前暂缓区的文件到github

```powershell
git push -u origin master
```

这条命令执行完之后，就可以在github上看到提交或者改动的代码了

