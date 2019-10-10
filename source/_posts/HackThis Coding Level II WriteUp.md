---
title: HackThis Coding Level II WP
date: 2019-03-04 15:27:12
tags:
- CTF
- Python
- 密码学
categories: WriteUp
---
# 题目

给出了一组按照某种编码方法编码后的数字，要求五秒内提交解码后的原文。加密方法如下

![](https://i.loli.net/2019/03/04/5c7cbe61bdad6.png)
<!--more-->
# 解题思路

## 编码方式分析
编码流程如下图所示：
![](https://i.loli.net/2019/03/04/5c7cbe3e1ce69.png)
![](https://i.loli.net/2019/03/04/5c7cbe4cc0420.png)


1. 将ASCII表中每个字符对应数字减去31
2. 在新表中找到待编码字符x对应的新编号x1
3. 用新表长度减去x1，找到对应位置上的字符x2
4. 在旧表中找到x2对应的编号x3，记录下来。

## 构造解码函数
在知道编码方式后就可以很简单的反推出解码的函数。
```
x=96-(x3-31)+31
```

## 编写爬虫

题目要求五秒内解决，所以就要使用python爬虫实现，本次我使用了requests库来实现。

利用session进行持久化访问，再通过post方法提交最后的答案，成功解决这道题。

# 后记

这道题难度不大，主要难在看懂编码方式上。看懂了编码方式，解题就变得很简单了。
最后我把我代码放在下面的图片里了，来做道我出的隐写题吧~
![](https://i.loli.net/2019/03/04/5c7cbe6ec3055.png)
如果挂了请用这个[链接](https://pan.baidu.com/s/1-05_A5Cm7E_fVWm4vrqFQg)
提取码: AWSL
hint：Find it out and get it fixed.