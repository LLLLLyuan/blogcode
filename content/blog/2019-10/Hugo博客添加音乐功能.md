---
data: "2019-10-13"
publishdate: "2019-10-13"
lastmod: "2019-10-13"
draft: false
title: "给Hugo博客添加(网易云)音乐功能"
tags: ["blog","Go","hugo"]
series: ["我和我的blog"]
categories: ["RecordMblog"]
toc: true
summary: "用网易云音乐给博客添加音乐功能"
---

<center>
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=28828076&auto=1&height=32"></iframe>
</center>

## 前言

感觉如果有人来看我的博客，有点音乐:notes:是不是会更好一些，既然需求都来了，那么想想怎么实现吧！

不用看，这个主题肯定没有这个功能，又要自己来，唉，原来选择一个好的主题是多么明确的选择:sob:

幸好不难


## 提取网易云音乐的链接

- 进入[网易云音乐网页版](https://music.163.com/)，然后搜一首自己喜欢的歌:notes:（以我和我的祖国为例）

    ![网易云音乐搜索](/images/blog/2019-10/网易云音乐搜索.png)

- 选择一首~~没有版权保护~~`有外链`的

    ![点击外链](/images/blog/2019-10/点击外链.png)

- 选择插件样式

    ![选择尺寸](/images/blog/2019-10/选择尺寸.png)

- 点击`复制代码`，修改参数

```html
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5234243&auto=1&height=32"></iframe>
```
    - `width` ：宽度，范围：`260－510`

    - `height` ：高度，范围：`190-500`
    - `auto` ：是否自动播放，`1`是，`0`不是

## 两种办法添加音乐

### 第一种：将代码加到模板中

- 优点：一次更换，永久使用，之后每次不用再手动输入，且所有指定的页面都会有音乐，适合对某一个歌曲能听一万遍的人群

- 缺点：若想换音乐需要`修改源代码`，有种一荣俱荣，一损俱损的赶脚

- 方法：在指定的`模板文件`中插入相应的`代码`即可。比如我的

    ![模板添加](/images/blog/2019-10/模板添加.png)
    
    `<center>`为`html`语法，作用为`居中`，效果如下

    ![模板效果](/images/blog/2019-10/模板效果.png)

### 第二种，将代码加入到Markdown文件中

- 优点：~~我想插到哪里就插到哪里，我想插什么音乐就插什么音乐，我想什么时候换就什么时候换~~`方便更换，随意搭配，选择纠结症患者福音`:clap:

- 缺点：每次编写文章都要重新插入:musical_note:（当然你也可以偶尔不插入）

- 方法：在`.md`文件中的适合位置`(我一般选择文章开头)`，输入刚才的代码就行

```html
<center>
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5234243&auto=1&height=32"></iframe>
</center>
```
如图：

![markdown添加](/images/blog/2019-10/markdown添加.png)

当然！`Markdown`**支持**`html语法`

![markdown效果](/images/blog/2019-10/markdown效果.png)

OK，音乐功能现在解决了，赶紧部署上去试一波 :cn:
## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-13