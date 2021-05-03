---
title: 第一个Python小爬虫
date: 2021-05-03 14:30:24
tags:  爬虫
categories: Python
---
![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fq6wto6ki3j20xq0io46x.jpg)

[上一篇文章](http://mp.weixin.qq.com/s/p7-LBoG8T3lQ4cKj4lg5sg)学习了Python Requests的使用，其中博客中还包括了[正则表达式的使用](https://labradors.work/2018/04/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E6%8A%80%E6%9C%AF/)，本篇文章是对前几篇文章的实践，写一个猫眼电影TOP100的爬虫。包括从网页分析，正则编写，数据转换，数据保存。下面就来详细说明。

<!--more-->

## 分析网页

打开网站[猫眼电影TOP100](https://maoyan.com/board/4),打开开发者工具，查看网页源代码，找到列表代码，也可以直接看下边的代码。

```html
<dd>
    <i class="board-index board-index-4">4</i>
    <a href="/films/4055" title="这个杀手不太冷" class="image-link" data-act="boarditem-click" data-val="{movieId:4055}">
        <img src="//ms0.meituan.net/mywww/image/loading_2.e3d934bf.png" alt="" class="poster-default">
        <img alt="这个杀手不太冷" class="board-img" src="http://p0.meituan.net/movie/fc9d78dd2ce84d20e53b6d1ae2eea4fb1515304.jpg@160w_220h_1e_1c">
    </a>
    <div class="board-item-main">
        <div class="board-item-content">
            <div class="movie-item-info">
                <p class="name">
                    <a href="/films/4055" title="这个杀手不太冷" data-act="boarditem-click" data-val="{movieId:4055}">这个杀手不太冷</a>
                </p>
                <p class="star">
                    主演：让·雷诺,加里·奥德曼,娜塔莉·波特曼
                </p>
                <p class="releasetime">上映时间：1994-09-14(法国)</p>
            </div>
            <div class="movie-item-number score-num">
                <p class="score">
                    <i class="integer">9.</i>
                    <i class="fraction">5</i>
                </p>
            </div>

        </div>
    </div>
</dd>
```

我们要从列表中过滤出图片，标题，时间，主演，位置以及评分。分析网页我们得出正则表达式为

```tex
<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>.*?star.*?>(.*?)</p>.*?releasetime.*?>(.*?)</p>.*?integer.*?>(.*?)</i>.*?fraction.*?>(.*?)</i>.*?</dd>
```

## 编写爬虫

```python
import requests
import re
import json
import time


def get_one_page(url):
    headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
    }
    response = requests.get(url,headers=headers)
    if response.status_code == 200:
        return response.text
    return None


def parse_one_page(html):
    pattern = re.compile(
        '<dd>.*?board-index.*?>(.*?)</i>.*?data-src="(.*?)".*?name.*?a.*?>(.*?)</a>.*?star.*?>(.*?)</p>.*?releasetime.*?>(.*?)</p>.*?integer.*?>(.*?)</i>.*?fraction.*?>(.*?)</i>.*?</dd>', re.S)
    items = re.findall(pattern, html)
    for item in items:
        yield {
            'index': item[0],
            'image': item[1],
            'title': item[2].strip(),
            'actor': item[3].strip()[3:] if len(item[3]) > 3 else '',
            'time': item[4].strip()[5:] if len(item[4]) > 5 else '',
            'score': item[5].strip() + item[6].strip()
        }

def write_to_json(content):
    with open('result.json','ab') as f:
        f.write(json.dumps(content, ensure_ascii=False,).encode('utf-8'))


def main(offset):
    url = 'http://maoyan.com/board/4?offset='+str(offset)
    html = get_one_page(url)
    for item in  parse_one_page(html):
        print(item)
        write_to_json(item)
        

if __name__ == '__main__':
    for i in range(10):
        main(offset=i*10)
        time.sleep(1)

```

我们一步步分析上面的代码

- `get_one_page:` 根据url爬取网页源代码。
- `parse_one_page:` 根据正则表达式，从网页中得到我们想要的内容。
- `write_to_json:` 将得到的数据写入文本。
- `main:` 循环获取网页数据。因为总共有一百条数据，而页面内容根据url路径改变。

## 结果

```json
{
    "time": "1993-01-01(中国香港)",
    "image": "http://p1.meituan.net/movie/20803f59291c47e1e116c11963ce019e68711.jpg@160w_220h_1e_1c",
    "title": "霸王别姬",
    "score": "9.6",
    "index": "1",
    "actor": "张国荣,张丰毅,巩俐"
} {
    "time": "1994-10-14(美国)",
    "image": "http://p0.meituan.net/movie/__40191813__4767047.jpg@160w_220h_1e_1c",
    "title": "肖申克的救赎",
    "score": "9.5",
    "index": "2",
    "actor": "蒂姆·罗宾斯,摩根·弗里曼,鲍勃·冈顿"
}
```

至此我们的第一个小爬虫算是完事了。明天尝试抓雪球首页新闻数据。后续可以尝试着抓取大佬的实盘操作，当对方股票，仓位有任何调整就第一时间发送通知到手机，及时进行操作。

-----
> 欢迎**长按下图 -> 识别图中二维码**或者微信**扫一扫**关注我的公众号
> ![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpzuv3q8ayj20w60ea11n.jpg)

