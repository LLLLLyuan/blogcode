---
data: "2019-10-16"
publishdate: "2019-10-16"
lastmod: "2019-10-16"
draft: false
title: "知乎爬虫(一)——requests篇"
tags: ["Python","爬虫"]
series: ["爬虫踩坑记"]
toc: true
summary: "量子力学法教你爬取知乎数据"
---

最近有关于"量子波动速读"的方法，不知道大家知道不知道。

传说只要学会了，来自宇宙的神秘力量就能让你拥有一种神奇的能力————**用1-5分钟，一直刷新本页面，你就能学会爬取知乎数据，并且是可以把代码内容完整还原的那种！**

至于你信不信，反正我是不信，我在想这年头这种忽悠人的法子都能赚钱，我要不要转行？

言归正传，今天记录一下前段时间写的``知乎爬虫``，用`requests`写个教程。

## 环境准备

`Python 3.7`（本人版本）

`Pypi` ：*requests* 、*pandas*

```bash
$ pip install requests pandas
```

## 分析

孔子曰：不服就干，正面刚！

于是乎，很多人就去[知乎首页](https://www.zhihu.com/)搞人家的验证码了，麻烦不麻烦，封你号都不亏！

[从这个入口进](https://www.zhihu.com/search?type=content&q=)，初极狭，才通人，复行数十步，豁然开朗！

嗯。。随便点个热搜词，国际惯例，用Chrome按`F12`后点`network`，`刷新下页面`，看看数据是怎么加载的。

嘿嘿，找到了！:laughing:

![F12](/images/blog/2019-10/F12.png)

很简单嘛，用post-man`不带`*header*看下

结果很正常!

再用post-man的一键生成代码，用`requests`访问试试

![response](/images/blog/2019-10/response.png)

这么少？感觉不太对，一看源代码发现，好家伙，数据是`json/text类型`返回的，而且都混在一起放在`<script>`标签里！

既然是`js`加载出来的，那么就一定有返回给他的`json数据`,下拉页面，知乎会继续用瀑布流加载数据，然后会发现另一个 url !

![zhihu_api](/images/blog/2019-10/zhihu_api.png)

难道这就是传说中的 `A`——`P`——`I` !

知乎api都找出来了，后面的事情就容易多了，继续post-man，不带header，请求正常。继续一键copy，生成代码

```python
url = "https://www.zhihu.com/api/v4/search_v3"
headers = {
        'User-Agent': "PostmanRuntime/7.18.0",
        'Accept': "*/*",
        'Cache-Control': "no-cache",
        'Host': "www.zhihu.com",
        'Accept-Encoding': "gzip, deflate",
        'Connection': "keep-alive",
        'cache-control': "no-cache"
    }
querystring = {"t": "general",
               "q": "英雄联盟手游",
               "correction": "1",
               "offset": "20",
               "limit": "20",
               "lc_idx": "26",
               "show_all_topics": "0",
               "search_hash_id": "9cbfd9be48e4c5fde0bfa668283066cd",
               "vertical_info": "0%2C1%2C1%2C0%2C0%2C0%2C0%2C0%2C1%2C1"
               }
response = requests.request("GET", url, headers=headers, params=querystring)
```

继续调试发现，有几个参数是可以改动的：

- `t`: 控制爬取栏目，
- `q`: 搜索关键词
- `correction` : 0的话会把热点信息加上，1没有
- `offset` : 相当于起始页，
- `limit` : 每页显示数据条数
- `lc_idx` : 会和`offset`一起变化，每次增加数量为上一页的`offset`数量，去掉没啥影响
- `show_all_topics` : 显示所有话题，没啥用，可以砍掉
- `search_hash_id` : 斩
- `vertical_info` : 也斩

换个UA，最后请求部分代码长这样

```python
url = "https://www.zhihu.com/api/v4/search_v3"
headers = {
    'User-Agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36",
    'Accept': "*/*",
    'Cache-Control': "no-cache",
    'Host': "www.zhihu.com",
    'Accept-Encoding': "gzip, deflate",
    'Connection': "keep-alive",
    'cache-control': "no-cache"
}
querystring = {"t": "general",
               "q": "英雄联盟手游",
               "correction": "0",
               "offset": "20",
               "limit": "20",
               }
response = requests.request("GET", url, headers=headers, params=querystring)
```
那么问题来了，如何一次性让爬虫去批量的爬呢？

有两个方法：

- 1、我们可以发现json中`next`的值就是 `下一页的url` ，如果此页是最后一页的话，`isfeed`的值是 *True* ，如果不是最后一页的话就是 *false* 。

    ![下一页](/images/blog/2019-10/下一页.png)

    思路：
    - 请求第一次，判断是否有下一页，保存返回结果并获取下一次的url    
    - 循环请求下一页，判断是否有下一页，保存返回结果并获取下一次的url
    - 请求到最后一页没有下一页url，保存返回结果，存储数据

    代码片段：
    ```Python
    class ZhihuSpider():
    
        def __init__(self,search_key,file_path):
            response = self.get_response(search_key)
            dict_data = self.json_to_dict(response.text)
            result_list = []
            url, self.result_list = self.get_result(dict_data,result_list)
            while url:
                response = requests.get(url, headers=self.headers)
                dict_data = self.json_to_dict(response.text)
                url, result_list = self.get_result(dict_data, self.result_list)
            self.sava_to_excel(file_path)
    ```
- 2、循环改变`offset`和值，批量构成url。

    这个很简单，先循环构造url列表，然后循环用requests请求，可以自己实现一下。

相比两个方法，第一个方法第一次写的慢些，后面如果还使用的话会很快，第二个方法虽然很快捷，但是在不知道数据量究竟多少的情况下，直接写死有点太`莽`了。（虽然我一开始用的第二种方法:joy_cat:）

## 完整代码
最后附上完整代码
```python
class ZhihuSpider():

    def __init__(self,search_key,file_path):
        response = self.get_response(search_key)
        dict_data = self.json_to_dict(response.text)
        result_list = []
        url, self.result_list = self.get_result(dict_data,result_list)
        while url:
            response = requests.get(url, headers=self.headers)
            dict_data = self.json_to_dict(response.text)
            url, result_list = self.get_result(dict_data, self.result_list)
        self.sava_to_excel(file_path)

    def get_response(self, search_key):
        url = "https://www.zhihu.com/api/v4/search_v3"
        self.headers = {
            'User-Agent': "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36",
            'Accept': "*/*",
            'Cache-Control': "no-cache",
            'Accept-Encoding': "gzip",
            'Connection': "keep-alive",
            'cache-control': "no-cache"
        }
        querystring = {"t": "general",
                       "q": search_key,
                       "correction": "1",
                       "offset": "20",
                       "limit": "20",
                       }
        response = requests.request("GET", url, headers=self.headers, params=querystring)
        response.encoding = response.apparent_encoding
        return response

    def json_to_dict(self, json_data):
        dict_data = json.loads(json_data, encoding="utf_8_sig")
        return dict_data

    def get_result(self, dict_data, result_list):
        if dict_data['paging']['is_end'] is False:
            next_url = dict_data['paging']['next']
        else:
            next_url = None
        for one in dict_data['data']:
            try:
                if one["type"] == "search_result":     # 知乎有时候会有其他推荐，类型不是search_result
                    one_info = {}
                    title = one["highlight"]["title"].replace("<em>", "").replace("</em>", "").strip()
                    question_id = one["object"]["question"]['id']
                    answer_id = one["object"]['id']
                    description = one["object"]['excerpt'].replace("<em>", "").replace("</em>", "").strip() + "..."
                    href = f"http://www.zhihu.com/question/{question_id}/answer/{answer_id}"
                    create_date = one["object"]['created_time']
                    one_info['url'] = href
                    one_info['create_date'] = datetime.fromtimestamp(create_date)
                    one_info['title'] = title
                    one_info['description'] = description
                    result_list.append(one_info)
                    print(one_info)
            except Exception as e:
                # print(str(e))
                pass
        return next_url,result_list

    def sava_to_excel(self, file_path):
        df = pandas.DataFrame(self.result_list)
        df.to_csv(file_path, index=False, encoding='utf-8')
        print("save ok")

if __name__ == '__main__':
    search_key = "英雄联盟手游"
    file_path = "zhuhu.csv"
    ZhihuSpider(search_key=search_key,file_path=file_path)
```
代码文件在[我的github](https://github.com/LLLLLyuan/zhihuspider)上，不懂来问

如果你遇到下图类似的乱码问题

![乱码](/images/blog/2019-10/乱码.png)

> 后面再告诉你怎么解决，嘿嘿：）

最后来个效果图

![csv](/images/blog/2019-10/csv.png)

## 版本控制
Version|Action|Time
:-:|:-:|:-:
1.0|first commit|2019-10-15
