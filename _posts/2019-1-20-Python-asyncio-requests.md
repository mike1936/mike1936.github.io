---
layout: post
title: Python asyncio requests 异步爬虫
author: Mike
keyword: Python
tags: #python #asyncio #requests #async/await #crawler
excerpt: 如何抓取大量URL（其中每个URL内含的信息量较少）
postid: 6
---
## 一、情景：
抓取**大量**URL，每个**URL内信息量较少**

> 任务清单: 发送URL请求N次，接受并处理URL响应N次


## 二、分析：
**① 如果每个页面依次抓取的话**：

> 任务流程：
发送第1条URL请求，接受并处理第1条URL响应，发送第2条URL请求，接受并处理第2条URL响应，发送第3条URL请求，接受并处理第3条URL响应……

时间会大量浪费在网络等待（IO-Bound）与执行网络请求命令（CPU-Bound）的切换上，且最重要的是，发出一个页面的网络请求（Request）后需要等待服务器回传信息，等待信息回传才发出下一个页面请求的话，不能高效地利用网络带宽；

**② 为每个页面抓取任务创建线程的话**：

> 任务流程：
线程一（并发）：发送第1条URL请求，接受并处理第1条URL响应；
线程二（并发）：发送第2条URL请求，接受并处理第2条URL响应；
线程三（并发）：发送第3条URL请求，接受并处理第3条URL响应；
……
线程N（并发）：发送第N条URL请求，接受并处理第N条URL响应）

线程之间的切换会造成大量的消耗


## 三、解决方法：

利用单个线程内的协程机制，异步执行所有任务清单中的任务。预先设定好需要抓取的URL的列表，触发所有URL页面请求，然后等待网络响应。利用python内置的asyncio调用requests（第三方库）实现异步抓取，提高效率。


## 四、实现：
#### 1. 预设想要抓取的URL的列表
（注：所有代码是连续的，依次拆分区块方便解释）
``` python3
#!usr/bin/env python3
# -*- code: utf-8 -*-

import asyncio
import functools
import os
import re

import requests

class MyRequest(object):
    def __init__(self):
        self.list = []
        make_list()
    def make_list(self, url):
        for i in range(1,1001):
            self.list.append('http://some.m3u8.play.list/{}.ts'.format(i))
```
#### 2. 抓取单个URL的协程编写
实际上就是编写一个生成器（Generator），然后利用**@asyncio.coroutine**将一个生成器标记/装饰（Decorate）为协程（Coroutine）；
在生成器中用**yield from**执行比较耗时的IO任务（在这里是网络传输的任务），并传回响应。

python 3.5之后的版本可以使用**async**与**await**关键词代替@asyncio.coroutine与yield from，让代码更加容易阅读。

``` python 3
async def crawler(url):
    print('Start crawling:', url)
    headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36'}
    
    # 利用BaseEventLoop.run_in_executor()可以在coroutine中执行第三方的命令，例如requests.get()
    # 第三方命令的参数与关键字利用functools.partial传入
    future = asyncio.get_event_loop().run_in_executor(None, functools.partial(requests.get, url, headers=headers))
    
    response = await future
     
    print('Response received:', url)
    # 处理获取到的URL响应（在这个例子中我直接将他们保存到硬盘）
    with open(os.path.join('.', 'tmp', url.split('/')[-1]), 'wb') as output:
        output.write(response.content)
```
#### 3. （可忽略）URL文件的后续处理
不过里边用到几个小技巧，可以看一下：

**① 文件夹遍历**
os.walk(path_to_go_through)分别返回路径的**根**（root），**文件夹类型**路径的列表（是相对路径，需要与root一起构成绝对路径）与**文件类型**路径的列表。

**② 按预设的序号排序**
sorted(list, key=index_function)返回排列好的list，list的排列方式依据list内每个元素的序号，而序号可以通过pass一个method到key参数来进行灵活设定。
*例如*：
`>>> list = ['file1', 'file10', 'file2', 'file20']`
`>>> sorted(list, key=lambda x : int(x[4:]))`
`['file1', 'file2', 'file10', 'file20']`

**③ 正则表达式的贪婪模式、匹配个数与分组**
- ? ----- 表示非贪婪匹配
- \* ----- 表示匹配任意个（包括零个）
- \+ ----- 表示匹配至少一个
- () ----- 分组（可以多次使用）：用括号括起的内容，若匹配到，可用.group(index)来调用，其中index=0时返回全部匹配，index=1时返回第一个分组，index=2时返回第二个分组……

**④ 文件路径的跨平台兼容**
使用os.path.join('folder', 'subfolder', 'file.txt')可以在不同平台下返回正确的文件路径
``` python3
def combine_files(input_folder, output_path, delete_origin=False):
    path_list = []
    # 遍历文件夹，寻找到所有类型为文件（而不是文件夹的）的路径
    for root, _, files in os.walk(input_folder):
        for file in files:
            path_list.append(os.path.join(root, file))
    # 合并所有响应文件为一个
    with open(output_path, 'wb') as output_file:
        for path in sorted(path_list, key=lambda x:int(re.match(r'.*?(\d+).ts', x).group(1)))
            with open(path, 'rb') as input_file:
                for line in input_file:
                    output_file.write(line)
    # 删除原始响应文件
    if delete_origin == True:
        for path in path_list:
            os.delete(path)
```
#### 4. 运行时命令
``` python3
if __name__ == '__main__':
    # 预先设定需要抓取的URL列表
    req = MyRequest()
    # 创建并执行协程任务
    loop = asyncio.get_event_loop()
    tasks = [crawler(url) for url in req.list]
    loop.run_untill_complete(asyncio.wait(tasks))
    loop.close()
    # URL响应文件的后续处理
    combine_files(os.path.join('.', 'tmp'), os.path('.', 'output', 'output.ts'), delete_origin=True)
```
## 五、后续需要完善的部分：
这篇文章只实现了多个网络请求IO的异步处理，之后需要研究一下如何在多个网络请求IO与本地存储（ROM/RAM）进行协调操作。

其他参考资源：

[1. Python遍历文件夹的两种方法比较](http://blog.51cto.com/laocao/525140)

[2. StackOverflow - How does asyncio actually work](https://stackoverflow.com/questions/49005651/how-does-asyncio-actually-work)

[3. StackOverflow -What do the terms “CPU bound” and “I/O bound” mean?](https://stackoverflow.com/questions/868568/what-do-the-terms-cpu-bound-and-i-o-bound-mean)