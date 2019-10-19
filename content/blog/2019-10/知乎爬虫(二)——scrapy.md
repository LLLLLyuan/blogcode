---
data: "2019-10-19"
publishdate: "2019-10-19"
lastmod: "2019-10-19"
draft: false
title: "知乎爬虫(二)——Scrapy篇"
tags: ["Python","爬虫","Scrapy"]
series: ["爬虫踩坑记"]
toc: true
summary: "真相只有一个，那就是..."
---

这篇教大家把用requests写的爬虫改装成Scrapy框架的爬虫。

## 环境准备

`Python 3.7`（本人版本）

`Pypi` ：*scrapy*

```bash
$ pip install scrapy
```

## 什么是Scrapy

Scrapy是目前Python使用的最广泛的爬虫框架，简单上手，稳定性及可扩展性高，并且使用此框架可以快速实现分布式爬虫（这篇不教哈哈）。

## Scrapy框架组成

先上个官方图

![scrapy](/images/blog/2019-10/scrapy.png)

看不懂没关系，我们一个一个看就能看懂了！

**`Scrapy Engine`**：引擎，责控制数据流在系统中所有组件中流动，并在相应动作发生时触发事件，它是整个框架核心。

**`Scheduler`**：调度器，从引擎接受request并将他们入队，以便之后引擎请求他们时提供给引擎。

**`Downloader`**：下载器，负责获取页面数据并提供给引擎，而后提供给spider。

**`Spiders`**：编写的爬虫文件， 每个spider负责处理一个特定(或一些)网站。

**`Item Pipeline`**：Item管道，负责处理被spider提取出来的item。典型的处理有清理、 验证及持久化(例如写成文件或者存储数据库)。

**`Downloader middlewares`**：下载器中间件，是在引擎及下载器之间的特定钩子(specific hook)，处理Downloader传递给引擎的response。 其提供了一个简便的机制，通过插入自定义代码来扩展Scrapy功能。说白了就是可以自定义代码处理response。

**`Spider middlewares`**：Spider中间件，是在引擎及Spider之间的特定钩子(specific hook)，处理spider的输入(response)和输出(items及requests)。说白了就是可以自定义代码处理Item和requests和response。

那么他的运作原理是什么呢？我们继续看，图中有好多`绿色的箭头`，他们是`数据流(Data flow)`。

Scrapy中的**数据流**由执行引擎控制，其过程如下:

1、 引擎打开一个网站(open a domain)，找到处理该网站的Spider并向该spider请求第一个要爬取的URL(s)。

2、引擎从Spider中获取到第一个要爬取的URL并在调度器(Scheduler)以Request调度。

3、引擎向调度器请求下一个要爬取的URL。

4、调度器返回下一个要爬取的URL给引擎，引擎将URL通过下载中间件(请求(request)方向)转发给下载器(Downloader)。

5、一旦页面下载完毕，下载器生成一个该页面的Response，并将其通过下载中间件(返回(response)方向)发送给引擎。

6、引擎从下载器中接收到Response并通过Spider中间件(输入方向)发送给Spider处理。

7、Spider处理Response并返回爬取到的Item及(跟进的)新的Request给引擎。

8、引擎将(Spider返回的)爬取到的Item给Item Pipeline，将(Spider返回的)Request给调度器。

9、(从第二步)重复直到调度器中没有更多地request，引擎关闭该网站。

简单来说就是，每次爬虫的requests请求都先给调度器，调度器再将这些url请求分配给下载器，下载器请求完后将response返回给爬虫，爬虫将response中提取出想要的信息给Item管道，最后由管道完成存储等操作。

这就是简单的一次循环，如果spider提取response时又让它提取出新的url请求，那么就会返回给调度器，进行下一次循环。

说了那么说，先拿上次用requests写的知乎爬虫试试手吧

## 创建及使用Scrapy

### 创建Scrapy项目

创建Scrapy项目步骤分三步：1、创建项目 2、进入项目文件夹 3、创建爬虫文件

创建项目命令：scrapy startproject <项目名称>

创建爬虫命令：scrapy genspider <爬虫文件名>  <爬虫域名>

以知乎爬虫为例
```bash
$ scrapy startproject zhihu

$ cd zhihu 

$ scrapy genspider zhihu_spider zhihu.com
```
命令输入成功后，`zhihu`项目文件夹里面会有以下几个文件

```text
.
├── zhihu
|   ├── __init__.py
|   ├── items.py
|   ├── middlewares.py
|   ├── pipelines.py
|   ├── settings.py
|   └── spiders
|      |── __init__.py
|      └── zhihu_spider.py
├── scrapy.cfg
```
此时，我们距离成功就差一点点(~~大杯冰淇淋红茶~~)

### 配置items.py文件

进入`items.py`后，编写以下代码
```python
class ZhihuItem(scrapy.Item):
    url = scrapy.Field()
    create_date = scrapy.Field()
    title = scrapy.Field()
    description = scrapy.Field()
    keyword = scrapy.Field()
```
其中，`url`、`create_date`、`title`、`description`、`keyword`就是将要存储的字段名称。

### 编写爬虫
`zhihu_spider.py`文件创建好后，大概像下面这个样子
```python
class ZhuhuSpider(scrapy.Spider):
    name = 'zhihu_spider'
    allowed_domains = ['zhihu.com']
    start_urls = []
    
    def parse(self, response):
        pass
```
`name`：爬虫名称，类型为 *str* 。
`allowed_domains`：域名限制，可以多个,类型为 *list* 。
`start_urls`：起始url，类型为 *list*。

爬虫在接受到下载器的response后，都会返回给这个`parse`函数。

>**parse** 函数名称不可更改！

将requests更改为scrapy后的完整代码如下

```python
# -*- coding: utf-8 -*-
import json
import re
from datetime import datetime
import scrapy
from scrapy import Request
from zhihu.items import ZhihuItem
from urllib import parse

class ZhihuSpider(scrapy.Spider):
    name = 'zhihu_spider'
    allowed_domains = ['zhihu.com']
    start_urls = []
    search_keys = ["雪莉", "S9", "海贼王狂热行动", "爬虫", "数据分析"]
    # 把想要搜索的搜索词加到search_keys里
    for search_key in search_keys:
        base_url = "https://www.zhihu.com/api/v4/search_v3"
        querystring = {"t": "general",
                       "q": search_key,
                       "correction": "1",
                       "offset": "20",
                       "limit": "20",
                       }
        url = f"{base_url}?{parse.urlencode(querystring)}"
        start_urls.append(url)
        # 将构建的url加入到start_urls里
        
    def parse(self, response):
        # print(response.status,response.url)
        search_key = re.search(r'q=(.*?)&', response.url).group(1)
        search_key = parse.unquote(search_key)
        # 转码html
        # print(parse.unquote(search_key))
        json_data = response.text
        dict_data = json.loads(json_data, encoding="utf_8_sig")
        if dict_data['paging']['is_end'] is False:
            next_url = dict_data['paging']['next']
            yield Request(url=next_url, callback=self.parse)
            # 在scrapy中，多个返回用yield而不用return
        for one in dict_data['data']:
            try:
                if one["type"] == "search_result":
                    item = ZhihuItem()
                    # 将定义过的Item容器传过来，类似字典
                    title = one["highlight"]["title"].replace("<em>", "").replace("</em>", "").strip()
                    question_id = one["object"]["question"]['id']
                    answer_id = one["object"]['id']
                    description = one["object"]['excerpt'].replace("<em>", "").replace("</em>", "").strip() + "..."
                    href = f"http://www.zhihu.com/question/{question_id}/answer/{answer_id}"
                    create_date = one["object"]['created_time']
                    item['url'] = href
                    item['create_date'] = datetime.fromtimestamp(create_date)
                    item['title'] = title
                    item['description'] = description
                    item['keyword'] = search_key
                    yield item
            except Exception as e:
                # print(str(e))
                pass
```

### 配置settings.py文件

```python
ROBOTSTXT_OBEY = False
DOWNLOAD_DELAY = 0.7
DEFAULT_REQUEST_HEADERS ={
    'accept-encoding': "gzip, deflate,br",
    'accept-language': "zh,en;q=0.9,q=0.8,zh-CN;q=0.7",
    'user-agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36",
}
DOWNLOADER_MIDDLEWARES = {
   'zhihu.middlewares.ZhihuDownloaderMiddleware': 543,
}

ITEM_PIPELINES = {
   'zhihu.pipelines.ZhihuPipeline': 300,
}
FEED_EXPORT_ENCODING = 'utf-8-sig'
```
`ROBOTSTXT_OBEY`：是否遵循robots协议，默认为*True*，将不会爬取有robots协议规定的网站，一般设置为 *False* 。 

`DOWNLOAD_DELAY`：下载延迟，就是请求间隔，为了减少对网站服务器压力，可根据实际情况调整，默认 *0.7* 。

`DEFAULT_REQUEST_HEADERS`：取消注释，并添加个通用headers，这样所有请求都会带上这个headers。

`DOWNLOADER_MIDDLEWARES`：开启 middlewares ,当有多个middleware时后面的值越高，优先度越高，默认 *543* 。

`ITEM_PIPELINES`：开启 pipeline ，后面的值越高，优先度越高，默认 *300* 。

`FEED_EXPORT_ENCODING`：导出文件的编码 。
### 运行Scrapy爬虫

运行命令：*scrapy crawl <爬虫名称>*

切换到与`scrapy.cfg`文件同目录下，运行下列代码

```bash
$ scrapy crawl zhihu_spider
```
命令输入完后回车，就可以看到数据会被输出到终端上了。

到这里大家可能发现了，没有数据存储！哈哈，这个步骤大家可以去试试怎么实现，我提供一个更简单的方法

```bash
$ scrapy crawl zhihu_spider -o zhihu_spider.csv
```
前面和运行命令的一样，多了一个参数 `-o File` 为 `--output=File` 的缩写

运行上述命令，即可看到项目文件目录下多了个 `zhihu_spider.csv` 文件！

此时，一个简单的scrapy项目已经完成:punch:

代码文件在[我的github](https://github.com/LLLLLyuan/zhihuspider)上，不懂来问


## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-19
