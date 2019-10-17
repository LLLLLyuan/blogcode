---
data: "2019-10-15"
publishdate: "2019-10-15"
lastmod: "2019-10-15"
draft: false
title: "什么是Brotli压缩"
tags: ["前端","前端性能","压缩算法"]
series: ["前端小知识"]
categories: ["我不会前端吖"]
toc: true
summary: "知己知彼，所向无敌"
---
<center>
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5248615&auto=1&height=32"></iframe>
</center>

准备写这篇文章的时候已经23点27分了，不是很困，回想起晚上遇到的一个bug，记录下我和`Brotli`的孽缘

## 前言 —— 什么是Brotli ？

Brotli是一个Jyrki Alakuijala和Zoltán Szabadka开发的开源数据压缩程序库。Brotli基于LZ77算法的一个现代变体、霍夫曼编码和二阶上下文建模。

在Chrome、Opera和Firefox中，它已被用于加速万维网的传输速度。类似Google的压缩算法zopfli，brotli这个名字来自瑞士的烘培产品brötli。

以上内容全部来自于[维基百科](https://zh.wikipedia.org/wiki/Brotli)。

一言以蔽之 ———— `一种用于web传输的压缩算法`。

## 有什么用 ?

也许前端开发对 `Brotli` 比较少见，但是对 `deflate`、`gzip` 压缩肯定是知道的。

一个网站用 gzip 和不用 gzip 效果很明显，打开速度能提高`30%`以上！

而 Brotli 又能保持与 gzip 差不多的解压速度，而且还比`gzip`多`20%`左右的压缩率

就是说它好呗

前端生产环境中将js、css、图片等文件进行压缩的好处显而易见，通过减少数据传输量减小传输时间，节省服务器网络带宽，提高前端性能。

就好比我的博客打开很慢，就是因为没有用压缩 :pig:

大部分浏览器都支持 `Brotli` 算法，除了`IE` 

所以到底有啥用？—————— **省流量啊**:pig:

## 如何判断网页是否支持 Brotli

很简单，一般用`Chrome`或者`Firefox`按`F12`打开`开发者工具`，切换到`Network`，刷新页面，点击一个加载出来的页面，然后查找一个参数 `content-encoding`

如果显示的是
```html
content-encoding: br
```
那么这个网页就是用的 brotli
- 浏览器请求数据时，通过 Accept-Encoding 说明自己可以接受的压缩方法

- 服务端接收到请求后，选取 Accept-Encoding 中的一种对响应数据进行压缩

- 服务端返回响应数据时，在 Content-Encoding 字段中说明数据的压缩方式

- 浏览器接收到响应数据后根据 Content-Encoding 对结果进行解压

> 如果服务器没有对响应数据进行压缩，则不返回 Content-Encoding，浏览器也不进行解压

一个大的网站，肯定是要使用压缩方式的，不然它的``性``~~`功`~~``能``有大大的问题

目前我所知道的网站压缩方式为 brotli 的有：[Google](https://www.google.com/) 、 [Bing](https://www.bing.com/) 、[知乎](https://www.zhihu.com)

为什么我知道呢？因为我写爬虫遇到了坑:shit: 

所以为什么我和`Brotli`有一段孽缘呢，因为它让我又爱又恨

好久没写爬虫了，准备抽个时间写个`知乎爬虫`的教程

听着摇篮曲，有点困了。。。

嗯，Brotli ，我们下次见👋



## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-15
2.0|change title error|2019-10-16

