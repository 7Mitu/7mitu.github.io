---
title: HackTheBox petpet rcbee WriteUP
date: 2023-08-23 12:11:49
tags:
- HackTheBox
- Web
- 漏洞利用
- CVE-2018-16509
---

# 题目

> Bees are comfy 🍯
bees are great 🌟🌟🌟
this is a petpet generator 👋
let's join forces and save the bees today! 🐝

![Bee](image.png)

![Web](image-1.png)

# 思路

`flag`文件的位置已经给出了，那么目标很明确，就是想办法读取到这个文件的内容。

代码中只有一个上传功能，python的站点通过上传木马GetShell显然是比较罕见的操作，那么更有可能的解题思路是对组件的漏洞进行利用。

![routers.py](image-2.png)

`DockerFile`中写入了安装`Pillow+ghostscript`的命令：

![dockerfile](image-3.png)

关于这两个组件的介绍，我们可以问一下神奇的ChatGPT：

![ChatGPT](image-4.png)

`ghostscript-9.23`存在远程命令执行漏洞，漏洞详情参考：[Python PIL/Pillow Remote Shell Command Execution via Ghostscript CVE-2018-16509](https://github.com/farisv/PIL-RCE-Ghostscript-CVE-2018-16509)

对代码进行分析后可以发现，当前的上传功能完美符合漏洞的利用条件。那么接下来，只需要找到`flag`文件的绝对路径就可以了。这很好找：

![flag_path](image-5.png)

路径是`/app/flag`

## 解题

读取`/app/flag`内容并写入到可访问的静态路径下。

![poc](image-6.png)

然后直接访问写入的文件即可。

![flag](image-7.png)

# 后记

~~开始的时候找错了绝对路径，还以为思路错了。~~