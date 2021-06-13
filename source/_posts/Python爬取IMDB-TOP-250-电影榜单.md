---
title: Python爬取IMDB TOP 250 电影榜单
date: 2019-06-30 19:05:08
categories: Python
tags: 爬虫
---

> “互联网电影资料库（英语：Internet Movie Database，简称IMDb）是一个关于电影演员、电影、电视节目、电视艺人、电子游戏和电影制作小组的在线数据库。”

IMDB TOP 250收录了世界上排名最高的250部电影，接下来写一个爬虫把这些电影收录起来。

首选语言当然是用世界上最流行的编程语言---Python了。

我用到两个库：requests、openpyxl，前者主要用来处理网络请求，后者用来操作excel文件。

<!-- more -->

安装:

```bash
pip install requests
pip install openpyxl
```


这个页面就是top 250电影榜单了https://www.imdb.com/chart/top，

分析一下网页元素，可以把排名、电影名（英文）、上映年份这几项提取出来，写一个正则表达式：

```
  pat = r'<td class="titleColumn">\s*(.*)..*\s*.*\s*title=".*" >(.*)</a>.*\s*<span class="secondaryInfo">\((.*)\)</span>'
```


预保存一下数据
```python
  res = find_all_by_pat(pat, imdb_doc)
```


拿到这几项之后还需要中文的电影名，资料来源我找的是豆瓣，下面这个地址可以得到一个非异步加载的页面，这样的话虽然比直接拿纯数据（通过网站的api拿纯数据有点麻烦，即使拿到了也很快就会被ban掉）慢一点，但是简单方便
```
  url = 'https://www.douban.com/search?cat=1002&q=%s' % query_name
```

观察网页可以发现，影片名很容易出现在相关电影这一栏下的第一行，我试了一下不能保证100%正确，但在大多情况下是准确的，这个办法会出现匹配不了的情况，但这样情况不多，在接受范围之内，这样最起码比手动收集快了很多倍，分析一下源码，得到如下正则：
```python
  pat2 = r'qcat.*\s*.*>(.*?)\s*</a>'
```

接下来一边访问一边插入数据就行了





现在数据已经完整，最后一步，创建一个excel文件，简单设置一下样式
```python
wb = Workbook()
sheet = wb.active
sheet.column_dimensions['B'].width = 60
sheet.column_dimensions['C'].width = 25
```

插入数据：
```
for i in range(len(res)):
  for j in range(len(res[i])):
    sheet.cell(row=i + 1, column=j + 1).value = res[i][j]
```

最后保存文件：

```python
wb.save('imdb.xlsx')
```

打开文件，数据全部拿到了(*^▽^*)





附上完整代码：
```
import random
import re
import requests
from openpyxl import Workbook
from time import sleep


def find_all_by_pat(pat, string):
  res = re.findall(pat, string)
  return res


def get_html_doc(url):
  pro = ['122.152.196.126', '114.215.174.227', '119.185.30.75']
  head = {
    'user-Agent': 'Mozilla/5.0(Windows NT 10.0;Win64 x64)AppleWebkit/537.36(KHTML,like Gecko) chrome/58.0.3029.110 Safari/537.36'
  }
  resopnse = requests.get(url, proxies={'http': random.choice(pro)}, headers=head)
  resopnse.encoding = 'utf-8'
  html_doc = resopnse.text
  return html_doc


def get_douban_html(query_name):
  url = 'https://www.douban.com/search?cat=1002&q=%s' % query_name
  douban_search_res = get_html_doc(url)
  return douban_search_res


def get_chinese_name(pat, doc):
  res_list = find_all_by_pat(pat, doc)
  try:
    return res_list[1]
  except:
    return ' '


def get_director_name(pat, doc):
  res = find_all_by_pat(pat, doc)
  try:
    return res[0].split('/')[1]
  except:
    return ' '


if __name__ == "__main__":
  url = "https://www.imdb.com/chart/top"
  imdb_doc = get_html_doc(url)
  pat = r'<td class="titleColumn">\s*(.*)..*\s*.*\s*title=".*" >(.*)</a>.*\s*<span class="secondaryInfo">\((.*)\)</span>'
  res = find_all_by_pat(pat, imdb_doc)

  # pat1 = '>(.*)\s*</a>\s*<span class="ic-mark ic-movie-mark">可播放</span>'
  pat2 = r'qcat.*\s*.*>(.*?)\s*</a>'
  pat3 = '<span\s*class="subject-cast">(.*)</span>'
  for i in range(len(res)):
    doc = get_douban_html(res[i][1])
    chinise_name = get_chinese_name(pat2, doc)

    director_name = get_director_name(pat3, doc)

    res[i] = list(res[i])

    res[i].insert(1, chinise_name)
    res[i].insert(3, director_name)

    print(res[i])
    sleep(random.random() * 1.2)

  wb = Workbook()
  sheet = wb.active
  sheet.column_dimensions['B'].width = 60
  sheet.column_dimensions['C'].width = 25
  for i in range(len(res)):
    for j in range(len(res[i])):
      sheet.cell(row=i + 1, column=j + 1).value = res[i][j]
  wb.save('imdb.xlsx')
```

电影详细信息可以保存我的另一篇博客: [传送门](https://blog.doctoroyy.net/2018/12/10/IMDB%20TOP%20250/)