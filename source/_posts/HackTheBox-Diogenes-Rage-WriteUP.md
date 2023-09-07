---
title: HackTheBox Diogenes' Rage WriteUP
date: 2023-08-23 16:55:08
tags:
- HackTheBox
- Web
- 条件竞争
---


## 题目

> Having missed the flight as you walk down the street, a wild vending machine appears in your way. You check your pocket and there it is, yet another half torn voucher coupon to feed to the consumerism. You start wondering why should you buy things that you don't like with the money you don't have for the people you don't like. You're Jack's raging bile duct.

![challenge](image.png)

> ~~你是杰克汹涌的胆汁导管~~

## 思路

一开始看了组件以为是个JWT的漏洞，绞尽脑汁没有找到思路之后发现……

是个条件竞争。

淦。

存在漏洞的代码如下：

- database.js

![vuln](image-1.png)

## 解题

首先调用`/api/purchase`获取cookie

![cookie](image-2.png)

开足线程跑就完了

![run！！](image-3.png)

拿到Flag

![Flag](image-4.png)

## 后记

~~万万没想到有一天会因为网速拿不到flag~~
条件竞争我怕不是跑了一百次……

> 总之这是一个从头到尾充满槽点的Challenge