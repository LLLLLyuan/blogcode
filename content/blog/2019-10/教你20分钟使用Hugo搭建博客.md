---
data: "2019-10-11T00:21:14+08:00"
publishdate: "2019-10-11T00:22:14+08:00"
lastmod: "2019-10-11T00:22:14+08:00"
draft: false
title: "教你20分钟使用Hugo搭建自己的博客"
tags: ["blog","Go","hugo"]
series: ["我和我的blog"]
categories: ["RecordMblog"]
img: ""
toc: true
summary: "记录下自己搭建博客的过程，并写个教程教一下想撘博客的童靴"
---


## Hugo简介

Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

（如果有人问我为什么我的博客打开这么慢，对不起，我的博客只能有缘人才能打开）

开始之前，请记录下时间————途中任何报错信息解决时间不能算进去，下载时间也不能算！

现在开始！

## 环境搭建

所需工具：

- Hugo
- Git
- Github账号

git的安装以及github账号创建网上教程很多，这里不介绍了，这也不是这个教程的重点，不能本末倒置。我默认你全都弄好了^_^

### Mac安装Hugo

mac安装hugo相对比较简单，只需要一个命令就可以下载下来

#### 使用homebrew安装

- 1、 使用homebrew安装hugo
    ```
    brew install hugo
    ```
    未安装homebrew的点这里：[Homebrew](https://brew.sh/)，将页面上的代码copy到终端`重击` `Enter`键即可。
    
- 2、 查看hugo是否安装成功
    ```
    hugo version
    ```
    若出现下图类似代码则说明安装成功。

    ![hugo_version](/images/blog/2019-10/hugo_version.png)

### Windows安装Hugo

Windows安装有以下几个方法：

#### 使用Chocolatey安装

- 在`CMD`中使用Chocolatey命令
    ```
    choco install hugo -confirm
    ```

#### 从github上下载

 - 1、直接从[Hugo的Github](https://github.com/gohugoio/hugo/releases)上下载相关的windows版本（一般是64位的那个）
 
 - 2、将压缩包`解压`到一个`文件夹`中（最好与其他软件的文件夹放在一起）
 
 - 3、在系统变量里的`Path`变量中添加上述`文件夹`路径,如`D:\hugo`

 > path添加方法请移步[百度经验-windows系统如何设置添加环境变量](https://jingyan.baidu.com/article/47a29f24610740c0142399ea.html)
 > 添加完后需要重启电脑才生效

 输入`hugo version`检查是否安装成功

### Ubuntu安装Hugo

#### 使用apt安装

```
sudo apt-get install hugo
```
同样使用``hugo version``检查，确认好安装成功后接下来就进入正式的创建博客了！

## 创建站点

首先可以先`cd`到你想要让博客文件生成的目录下，然后执行下述命令
```
hugo new site myblog
```


## 下载主题

hugo的主题可以到[hugo官方主题库](https://themes.gohugo.io/)里找，不过说实话都不怎么好看。

> 我用的主题为[AllinOne](https://github.com/orianna-zzo/AllinOne)

找到一款你喜欢的主题，比如主题`hyde`吧

- 1、找到相关主题的Github地址，比如hyde主题的地址为`https://github.com/spf13/hyde.git`

- 2、在`myblog`目录下创建个`themes`文件夹（存在就不需要创建）
- 3、`cd`到`themes`文件下里，然后把主题`clone`下来
```
cd themes

git clone https://github.com/spf13/hyde.git
```
clone完毕后`themes`文件夹里就会有一个`hyde`的文件夹


## 启用主题

- 回到`myblog`目录下，启用主题看下效果

```
cd ..

hugo server --theme=hyde --buildDrafts
```
此时会看到一段提示是`http://localhost:1313/`输入这个地址到浏览器中就可以看到效果了

## 创建文章 (不是那个文章啦）

此时可以先关闭一下博客服务（ctrl + c），然后使用`hugo`的`new`命令创建一篇文章
```
hugo new post/first_blog.md
```
hugo使用new命令会在content文件夹自动生成你创建的md文件。上述代码意思是在`content`文件夹下创建个`post`文件夹，再在`post`文件夹里创建`first_blog.md`文件，为什么要在post文件夹，因为方便管理~

然后可以打开`first_blog.md`文件，在里面随便写下内容，然后重新使用命令启动博客服务:
```
hugo server --theme=hyde --buildDrafts
```

然后重新打开`http://localhost:1313/`就可以看到你新写的文章已经生成好了！


接下来是部署的问题

## 部署github

### 创建Repository

- 1、`登录github`点击右上角的`+`，创建一个新的`repository`

- 2、设置`Repository name`
Repository name为你的`昵称.github.io`,比如我的github为`LLLLLyuan`，那么相应的repository name为`lllllyuan.github.io`（记得大写的通通要改为小写）。

![github_repository](/images/blog/2019-10/github_repository.png)

> 注意：答应我，不要任性地把`Repository name`的前缀改为其他的好吗


然后点击最下面的`Create respository`

### 部署

- 1、生成静态页面
在`myblog`文件目录下执行：
```
hugo --theme=hyde --baseUrl="http://lllllyuan.github.io/"
```
出现以下界面表示生成成功

    ![public](/images/blog/2019-10/public.png)

- 2、使用`git`，将生成的`public`文件上传到github上，以我自己的为例，依次执行如下命令
```
cd public
git init
git remote add origin https://github.com/lllllyuan/lllllyuan.github.io.git
git add .
git commit -m "first_blog commit"
git push -u origin master
```
    出现以下界面说明push成功
    ![push](/images/blog/2019-10/push.png)

**注意**：

- 将`https://github.com/lllllyuan/lllllyuan.github.io.git`中`lllllyuan`换成你的，地址就是你创建好的`repository`的地址后加`.git`（或直接在页面上点击`clone with https`的`复制按钮`）
![clone_with_https](/images/blog/2019-10/clone_with_https.png)

- 中间可能需要输入密码（第一次push的时候的需要）

- 记得要切到`public`文件夹下

此时算是大功告成了

结束！撒花~

用时`19分58秒`，管你信不信:ρ

之后就可以去你的github上刷新下，然后访问`lllllyuan.github.io`就可以看到你的博客了（可能需要等两分钟）


## 容易出问题的几个地方

- 1、homebrew安装过慢————找个梯子试试

- 2、git clone不下来————找个梯子试试

- 3、主题启动失败————~~找个梯子试试~~多半是主题的`参数`没有配置好，去主题地址看下`README`怎么使用的,然后将参数添加到`config.toml`文件里去

- 4、新建文章没有加载出来————有些主题`存放文章的文件夹`设置的名称不同，有些叫`post`，有些可能叫`blog`或者`wenzhang`，在相应的文件夹下创建即可

- 5、部署后发现文章没有生成————去掉文章头部（不是那个文章的头部）的 `draft=true`，或者将`true`改为`false`

- 6、博客打开慢或打不开————如果不是文件过大的话，这个可能真得找个梯子。

**btw:想买梯子的可以在我的[友情链接](https://bestyuan.fun/friendlylink/)里找[Sitio](https://www.sitoi.cn/)或者我买，便宜好用^_^**

## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-12