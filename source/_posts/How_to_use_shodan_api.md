---
title: 使用python在Shodan上进行批量查询
date: 2019-03-15 23:33:33
tags:
- 工具使用
- 信息收集
- Python
categories: 学习笔记
---
# 什么是Shodan
[Shodan](https://www.shodan.io)是一个搜索引擎，和Google类似，但它更像是一个dark♂版的谷歌。
![](https://i.loli.net/2019/03/15/5c8ba7230cb0e.png)

<!--more-->
它会搜索网络空间中的设备并记录设备的各种信息，包括设备可能包含的漏洞。Shodan支持多种形式的搜索，包括针对漏洞类型，针对目标属性，针对物理区域等，但是大部分搜索都需要氪金以~~变得更强~~获取全部信息。

在这里我们假设手中有了一系列待审计的IP列表~~加急名单~~，只针对特定的IP进行查询。

# 针对IP进行搜索
使用shodan的网页就能简单的获取到目标IP的信息：

这里用B站的IP为例，网站给出了IP的物理位置，使用的Web技术，开放的端口以及端口上运行的服务：
![](https://i.loli.net/2019/03/15/5c8ba6fc0380f.png)

对于一些有漏洞的IP，还会提供目标的漏洞名以及漏洞详细信息，比如下面这个地址：
![](https://i.loli.net/2019/03/15/5c8ba712bab95.png)

但是，如果我们的列表很长，没办法通过人工的方法对IP进行查询呢？

一种解决方法就是使用Python爬虫，利用requests库提交get请求完成查询。但是由于某不可抗力，查询速度十分有限。下面介绍一种更加快捷的方法。
# 利用Python连接Shodan提供的API
Shodan提供了一个Python库，只需要在命令行中输入
```
pip install shodan
```
就可以自动完成安装

在使用api前，还需要去shodan上注册一个个人账户，然后拿到你的API Key：
![](https://i.loli.net/2019/03/15/5c8ba6e857b36.png)

之后就可以愉快的写脚本啦~

对于Shodan模块在这里我不进行太详细的介绍，只要知道使用该模块的时候需要先把手里的API作为参数传入后利用.host()方法就能对IP进行查询就够了。

更多Shodan的API用法请参照[官方文档](https://developer.shodan.io/api)

下面是对IPList中的IP进行批量查询的代码

```python
import shodan

key='你的API Key'
api=shodan.Shodan(key)# 传入API key，开启API

IPList=['...']         # 读入的IP列表
resultList=[]     # 用于储存查询结果 
for i in IPList:
    resultList.append(api.host(i)) # 结果储存于resultList中
    print(i+' Done')
```
使用API进行查询的速度是利用网页爬虫进行爬取的6倍以上，实测利用API查询17,000+条记录仅需要五小时，而利用requests库进行网页爬虫大约需要30~40小时。

查询回来的数据会以字典的形式储存在列表里，如果想存为文件待日后慢慢分析可以使用pickle模块完成搜索结果持久化，但那就是另一篇~~水文~~博客了

# 总结：

最近需要用到这方面的知识就学习了一波，没想到有这么excited的网站。可惜毕竟是别人的数据，对方肯定会对数据进行过滤。要是想要可信度高的数据还是要靠自己进行扫描。

但是如果面对审计大批量IP的情况，利用此类网站进行快速的调查就是一种可以考虑的手段了。