---
title: 一个Get-Title的自我修养
date: 2021-01-26 16:31:49
tags:
- Python
- 爬虫
- 工具
---
# 0x00 前言

最近要整理大量的网页资料，刚好完善一下以前写的get-title脚本。
目标：获取所有URL对应网页的Title，并以友好的格式输出至文件。
此脚本原来是渗透的时候搞网段用的，把扫出来的Web的title列举出来，从而对自己的目标有个大致的概念。但是早先的版本只能说是可以将就着用，往往输出的格式乱七八糟，刚好借着这次机会重写一下。也顺便将其从python2过渡到python3。

# 0x01 版本对比

早期版本：

```python
import requests
from bs4 import BeautifulSoup
from threading import Thread
from Queue import Queue
import sys
import time
import signal
import chardet

def getTitle(line):
    try:

        line=line.strip()
        re = requests.get(line, timeout=2)
        ret2.write(line + '\n')
        print line+'\t\t'+str(re.status_code)

        if re.status_code==200:
            text=BeautifulSoup(re.content,'html.parser')
            titles=text.find('title')
            title=str(titles)
            ret.write(line+'\t\t'+title+'\n')
        else:
            ret.write(line + '\t\t' + 'Error Code:' + re.status_code + '\n')

    except requests.exceptions.Timeout:
        ret2.write(line + '\terror\n')
        print line+'\t\ttime out'
        ret.write(line+'\t\t'+ 'Time out\n')

class Worker(Thread):
    def __init__(self, taskQueue):
        Thread.__init__(self)
        self.setDaemon(True)
        self.taskQueue = taskQueue
        self.start()

    def run(self):
        while 1:
            try:
                callable, args, kwds = self.taskQueue.get(block=False)
                callable(*args, **kwds)
            except:
                break


class ThreadPool:
    def __init__(self):
        self.threads = []
        self.taskQueue = Queue()
        self.threadNum = num_thread
        self.__create_taskqueue()
        self.__create_threadpool(self.threadNum)

    def __create_taskqueue(self):
        f = open("target.txt", 'r')
        lines = f.readlines()
        for line in lines:
            self.add_task(getTitle, line)
        f.close()

    def __create_threadpool(self, threadNum):
        for i in range(threadNum):
            thread = Worker(self.taskQueue)
            self.threads.append(thread)

    def add_task(self, callable, *args, **kwds):
        self.taskQueue.put((callable, args, kwds))

    def new_complete(self):
        while 1:
            time.sleep(0.1)
            alive = False
            for i in range(num_thread):
                alive = alive or self.threads[i].isAlive()
            if not alive:
                break


def handler(signum, frame):
    global is_exit
    print "CTRL+C Is Pressed"
    sys.exit(0)


if __name__ == '__main__':
    num_thread = 20
    signal.signal(signal.SIGINT, handler)
    signal.signal(signal.SIGTERM, handler)
    ret = open("titles.txt", "w")
    ret2=open("test.txt",'w')
    tp = ThreadPool()
    tp.new_complete()
    ret.close()
    ret2.close()

```

重写极简版本：

```python
import requests
from bs4 import BeautifulSoup

res=requests.get(url)
soup=BeautifulSoup(res.text,'html.parser')
print(soup.title.string)

```

最终版本：

```python
import requests
from bs4 import BeautifulSoup
import threadpool
import re
from signal import signal, SIGINT
from sys import exit

proxies={'http':'http://127.0.0.1:10809'}
threads=30
timeout=3

def handler(signal_received, frame):
    print('SIGINT or CTRL-C detected. Exiting gracefully')
    exit(0)

def get_urllist(file):
    with open(file,'r') as target:
        targets=target.readlines()
        return targets

def get_title(url):
    headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36'}
    try:
        res=requests.get(url,headers=headers,proxies=proxies,timeout=timeout)
    except :
        return 'Timeout'
    if res.apparent_encoding != None:
        response=res.content.decode(res.apparent_encoding)
    else:
        response=res.text
    try:
        if 'mp.weixin.qq.com' in url:
            rule=r"var msg_title = '.*'"
            title=re.search(r"'.*'",re.search(rule,response).group()).group().strip('\'')
        else:
            soup=BeautifulSoup(response,'html.parser')
            if soup.title:
                title=str(soup.title.string)
            else :
                title=''
    except Exception as e:
        print(e)
        exit(0)
    return title

def single_thread(url):
    url=url.strip('\r\n')
    result=url + '\t' + str(get_title(url))
    print(result)
    with open('result.txt','a+',encoding='utf-8') as output:
        output.write(result+'\n')

if __name__ == '__main__':
    signal(SIGINT, handler)
    target=get_urllist('target.txt')
    pool = threadpool.ThreadPool(threads)
    threading=threadpool.makeRequests(single_thread,target)
    [pool.putRequest(req) for req in threading]
    pool.wait()
```

简而言之，早期的版本与当前版本区别如下：

- 利用多线程的方式有所区别
- 解决了不同网页编码格式不同的问题
- 增加了代理选项
- 解决了微信公众还title爬取不到的问题
- 解决一些其他的小BUG
- 一些使用体验上的优化

# 0x02 探索历程

早期的版本实际上是直接对其他大佬的代码做的修改，仅仅在使用习惯上做了一些调整，代码逻辑也不甚了解，于是一不做二不休，从零开始重写脚本。

## 坑1 微信公众号文章的Title

最早用极简版测试的时候，发现所有的微信公众号都无法用bs4直接获取到title，于是乎瞅了一眼公众号的源码，title竟然是这个屌样子的……

```html
<script>
    …………
    var hd_head_img = "http://xxxxxxxxxx/"||"";
    var ori_head_img_url = "http://xxxxxxxxx/";
    var msg_title = '这里是title'.html(false);
    var msg_desc = "XXXXXXXXXX...";
    var msg_cdn_url = "http://XXXXXX/..."; 
    …………
</script>
```

丧心病狂啊……这操作我没太看明白，防爬？

想多了吧。

直接正则一把梭：

```python
if 'mp.weixin.qq.com' in url:
    rule=r"var msg_title = '.*'"
    title=re.search(r"'.*'",re.search(rule,response).group()).group().strip('\'')
```

## 坑2 没有Title

有些链接是文件的下载链接，没有Title，于是引发bs4报错，于是引发脚本崩溃，这……

```python
soup=BeautifulSoup(response,'html.parser')
if soup.title:
    title=str(soup.title.string)
else :
    title=''
```

## 坑3 User-Agent被拦截

有的防护设备居然会丧心病狂的拦截requests的UA……

```python
headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36'}
try:
    res=requests.get(url,headers=headers,proxies=proxies,timeout=timeout)
except :
    return 'Timeout'
```

## 坑4 编码问题

这其实是个挺头疼的问题，之前第一版的脚本就一直没有解决。
以前读取`response`的内容，一般是通过两个方式，`res.text()`或者`res.content()`。但是这样有一个很头疼的问题，就是每个网站的编码方式不一样，尤其中文网站，用`GBK`的和用`UTF-8`的网站几乎一样多。于是输出的时候就是各种乱七八糟的乱码，而一个文件又不可能同时有两种编码格式。
经过艰(qing)难(jiao)研(da)究(lao)，最终确定了两种解决方案：

1. 通过`chardet`确定编码格式，最终统一成同一种编码方式；
2. 读取网页`response`头中的编码格式，然后decode；

最终我采用的是方案2：

```python
response = res.content.decode(res.apparent_encoding)
```

这里又有一个小坑，有些网站的response头中不会返回编码格式……

这类网站往往都是默认采用`UTF-8`格式编码，所以我们直接用`res.text`就可以了：

```python
if res.apparent_encoding != None:
    response=res.content.decode(res.apparent_encoding)
else:
    response=res.text
```

# 0x03 总结

这次脚本的编写还算比较顺利（毕竟是个很简单的东西），从开始到调试、完工也不过花了两个多小时，还不如写这篇文章花的时间多，大部分时间都花在了滤坑上面。但是其实还是可以分析出一些东西，一是本人确实久疏战阵，不太熟练了；二来，即便比早先的版本改进了一些，但距离作为一个成熟的工具，仍有许多可以改进的地方。

## 仍然存在的缺陷

1. 遭遇某些编码格式的网站时，仍然会报错（如`cp1254`等）；

```python
  File ".\get-title.py", line 28, in get_title
    response=res.content.decode(res.apparent_encoding)
  File "C:\Environment\Python38\lib\encodings\cp1254.py", line 15, in decode
    return codecs.charmap_decode(input,errors,decoding_table)
```

2. 对于一些比较常见的反爬虫手段，无能为力（爬到的title是`Just a moment...`，说明在自动验证是否真人访问）

## 可以改进的方向

1. 增加代理池模式，用以解决部分网站TimeOut的问题；
2. 更加友好的结果呈现，可输出至Excel表格中，最好舍弃csv采用xlsx，因为获取的title千奇百怪，可能破坏csv的格式；
3. 自动识别一些常见的中间件，如Weblogic等等；


# 0x04 2021.2.26更新

``` python
import requests
from bs4 import BeautifulSoup
import threadpool
import re
from signal import signal, SIGINT
from sys import exit
from sys import argv

use_proxy=True
proxies={'http':'http://127.0.0.1:10809','https':'http://127.0.0.1:10809'}
# proxies={'http':'http://127.0.0.1:10809'}
threads=30
timeout=10
result_encode_type='gb18030'

def handler(signal_received, frame):
    print('SIGINT or CTRL-C detected. Exiting gracefully')
    exit(0)

def get_urllist(file):
    with open(file,'r') as target:
        targets=target.readlines()
        return targets

def get_title(url):
    if 'http' not in url:
        url = 'http://'+url
    headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36'}
    try:
        if use_proxy:
            res = requests.get(url,headers=headers,proxies=proxies,timeout=timeout)
        else:
            res = requests.get(url,headers=headers,timeout=timeout)
    except Exception as e:
        print(e)
        return 'Timeout'
    if res.apparent_encoding != None:
        try:
            encode_type=res.apparent_encoding
            response=res.content.decode(encode_type)
        except:
            print("#Warning# Can't decode string as 【%s】.Target URL is 【%s】." % (res.apparent_encoding,url))
            response=res.text
    else:
        response=res.text
    try:
        if 'mp.weixin.qq.com' in url:
            rule=r"var msg_title = '.*'"
            title=re.search(r"'.*'",re.search(rule,response).group()).group().strip('\'')
        else:
            soup=BeautifulSoup(response,'html.parser')
            if soup.title:
                title=str(soup.title.string)
            else :
                title=''
    except Exception as e:
        print(e)
        exit(0)
    return title.strip('\r\n')

def single_thread(url):
    url=url.strip('\r\n')
    if not url:
        return
    result='"'+url + '","' + str(get_title(url))+'"'
    print(result)
    with open('result.csv','a+',encoding=result_encode_type) as output:
        output.write(result+'\n')

if __name__ == '__main__':
    if len(argv)!=2:
        print('Usage:\n  python3 get-title.py [targetfile]')
        exit()
    target_file=argv[1]
    signal(SIGINT, handler)
    target=get_urllist(target_file)
    pool = threadpool.ThreadPool(threads)
    threading=threadpool.makeRequests(single_thread,target)
    [pool.putRequest(req) for req in threading]
    pool.wait()
    
```

- 优化了输出方式，改为输出到CSV表格；
- 修正了爬HTTPS会出现问题的BUG，这么明显的BUG一开始的时候居然没发现……
- 修正了部分网站爬取时编码问题异常的BUG；