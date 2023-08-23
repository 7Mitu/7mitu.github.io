---
title: HackTheBox petpet rcbee WriteUP
date: 2023-08-23 12:11:49
tags:
- HackTheBox
- Web
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

`DockerFile`显示