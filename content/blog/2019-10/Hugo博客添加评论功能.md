---
data: "2019-10-13"
publishdate: "2019-10-13"
lastmod: "2019-10-13"
draft: false
title: "给Hugo博客添加utterances评论功能"
tags: ["blog","Go","hugo","前端"]
series: ["我和我的blog"]
categories: ["RecordMblog"]
toc: true
summary: "用hugo + utterances给博客添加评论系统"
---

## 前言

因为我选的这个主题没有评论系统，怎么办呢？主题是不可能换的，因为我看中这个[主题](https://github.com/orianna-zzo/AllinOne)了，作者也已经停止维护此主题一年多了。

既然如此，那就只能自己加了，就当顺便学一下前端咯。:punch:

因为是用Hugo搭建的博客，所以只看中了三个评论系统：1、`Disqus` 2、`Valine` 3、`Utterances`

说下三者区别：

- **Disqus** 
    - 优点：漂亮，hugo自带，[官方文档](https://gohugo.io/templates/internal/#configure-disqus)中有写如何启动，直接在`config.toml`文件中启用就可以用。
    - 缺点：广告多，国外的，容易被墙。
    
- **Valine**
    - 优点：国内，免费，不需要登录就能评论，速度快，只需要从[官方](https://valine.js.org/quickstart.html#HTML-%E7%89%87%E6%AE%B5)复制代码片段粘贴到模板中即可。
    - 缺点：处于开发阶段，开发者需要注册[LeanCloud](https://leancloud.cn/dashboard/login.html#/signup)账号，最重要的是~~`我没钱买服务器`~~需要**`网站已备案`**

- **Utterances** 
   - 优点：基于GitHub issues的评论工具，开源免费，加载快，配置较简单，没有广告，没有追踪（很多评论系统有追踪，感觉不是很尊重访客隐私）。
   - 缺点：较其他几个评论系统比较`吃藕`
   
好像三个都支持`Markdown`

## 环境准备

- **Github账号**
- **Github账号**
- **Github账号**

> 重要的事情说三遍

## 安装

- 先到[Github App](https://github.com/apps/utterances)里安装`utterances`

  ![utterances安装第一步](/images/blog/2019-10/utterances安装第一步.png)

  点击`install`

  ![utterances安装第二步](/images/blog/2019-10/utterances安装第二步.png)

  再点`install`

- 然后到自己的[github](https://github.com/)上创建一个`repository`，如果你看了我上一篇[教你20分钟使用Hugo搭建自己的博客](https://bestyuan.fun/blog/2019-10/%E6%95%99%E4%BD%A020%E5%88%86%E9%92%9F%E4%BD%BF%E7%94%A8hugo%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E5%8D%9A%E5%AE%A2/)的话，那么你已经有一个github的博客仓库了，你可以使用这个或者创建个新的。如何创建可以参考[这篇文章](https://bestyuan.fun/blog/2019-10/%E6%95%99%E4%BD%A020%E5%88%86%E9%92%9F%E4%BD%BF%E7%94%A8hugo%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84%E5%8D%9A%E5%AE%A2/)中的`创建Repository`部分。

## 配置代码

有两种方式添加，具体情况看个人（推荐第二种）

- 1、最简单但是不能够随意开启，不能个性化设置

    直接从[官方](https://utteranc.es/?installation_id=2919562&setup_action=install)或者从`下方`复制代码片段粘贴到`_default`里的``相应的.html``文件的相应位置即可

    ```html
    <script src="https://utteranc.es/client.js"
            repo="[ENTER REPO HERE]"
            issue-term="pathname"
            theme="github-light"
            crossorigin="anonymous"
            async>
    </script>
    ```
    其中`[ENTER REPO HERE]`填写格式为`owner/repo`,比如我的就是`LLLLLyuan/lllllyuan.github.io`
    ![utterances仓库设置](/images/blog/2019-10/utterances仓库设置.png)

    
- 2、模块化，可设置开启关闭，便于管理。

     在`themes`文件夹里的`你的主题`文件夹里`layouts`里的`partials`里创建个**comments.html**文件

    以我的为例就是``myblog/themes/AllinOne/layouts/partials/comments.html``,然后`复制`下面的代码`copy`进去，`保存`。

    ```html
    {{ if .Site.Params.utteranc.enable }}
    <div class="post bg-white">
      <script src="https://utteranc.es/client.js"
            repo= "{{ .Site.Params.utteranc.repo }}"
            issue-term="{{ .Site.Params.utteranc.issueTerm }}"
            theme="{{ .Site.Params.utteranc.theme }}"
            crossorigin="anonymous"
            async>
      </script>
    </div>
    {{ end }}
    ```
    其中`div`标签的`class`属性可以选择你已有的`class`，没有的话请置空为`<div>`

    然后在`config.toml`文件中添加相关的参数
    ```toml
    [params.utteranc]
    enable = true
    repo = "LLLLLyuan/lllllyuan.github.io"    # 存储评论的Repo，格式为 owner/repo
    issueTerm = "pathname"  #表示你选择以那种方式让github issue的评论和你的文章关联可以选择默认的pathname。
    theme = "github-light" # 样式主题
    ```
    然后再在``相关的模板文件``中添加一段代码`{{ partial "comments.html" . }}`即可
    ![添加评论](/images/blog/2019-10/添加评论.png)

## Utterances相关参数配置

- `repo` : 格式为`owner/repo`即`用户名/仓库名`，千万不要填错了

- `issue-term` : 映射配置,有以下几种设置可供使用(个人使用的pathname)
    - pathname
    - url
    - title
    - og:title
    - issue-number
    - specific-term
- `theme` : 也就是主题，一白五黑，我白色主题没得选:wave:
    - github-light
    - github-dark
    - github-dark-orange
    - icy-dark
    - dark-blue
    - photon-dark

本地启动hugo看一下效果吧:v:

## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-13