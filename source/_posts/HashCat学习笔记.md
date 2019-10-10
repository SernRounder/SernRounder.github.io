---
title: HashCat学习笔记
date: 2018-12-25 21:19:20
tags: 
- misc
- 密码学
- CTF
- 工具使用
categories: 学习笔记
---
# 简述
hashcat是一个极其强大的密码破解工具，在它的官网上有着这么一句话
>World's fastest password cracker

它通过使用GPU来加速计算hash的速度，从而达到快速破解密码的目的。经过测试，在我的电脑的配置情况下使用显卡计算WPA/WPA2密码的速度大约是仅用cpu的8倍~10倍。
<!--more-->

# 工作原理
从官方介绍中了解到hashcat通过利用OpenCL来使用GPU和CPU进行计算。
>GPUs are not magic superfast compute devices that are thousands of times faster than CPUs – actually, GPUs are quite slow and dumb compared to CPUs. If they weren't, we would even use CPUs anymore; CPUs would simply be replaced with GPU architectures. What makes GPUs fast is the fact that there are thousands of slow, dumb cores (shaders.) This means that in order to make full use of a GPU, we have to parallelize the workload so that each of those slow, dumb cores have enough work to do. Password cracking is what is known as an “embarrassingly parallel problem” so it is easy to parallelize, but we still have to structure the attack (both internally and externally) to make it amenable to acceleration. 
>For most hash algorithms (with the exception of very slow hash algorithms), it is not sufficient to simply send the GPU a list of password candidates to hash. Generating candidates on the host computer and transferring them to the GPU for hashing is an order of magnitude slower than just hashing on the host directly, due to PCI-e bandwidth and host-device transfer latency (the PCI-e copy process takes longer than the actual hashing process.) To solve this problem, we need some sort of workload amplifier to ensure there's enough work available for our GPUs. In the case of password cracking, generating password candidates on the GPU provides precisely the sort of amplification we need. In Hashcat, we accomplish this by splitting attacks up into two loops: a “base loop”, and a “mod(ifier) loop.” The base loop is executed on the host computer and contains the initial password candidates (the “base words.”) The mod loop is executed on the GPU, and generates the final password candidates from the base words on the GPU directly. The mod loop is our amplifier – this is the source of our GPU acceleration. 

以上是官方FAQ中的工作方式说明。简单的来说，CPU和GPU都有计算单元，CPU的计算单元功能强大但是数量少，GPU的计算单元非常“dump”但是数量庞大。计算密码这项工作可以被拆分到能够被GPU的计算单元处理，因而可以通过使用GPU的处理单元处理计算密码时产生的大量数据，从而大大提高计算效率。
# 安装
hashcat的官网上有详细的安装教程，我是直接下载了编译完成的二进制文件放在window环境中运行。官网上也提到了在Linux和windows下它的表现是相近的。

# 基础使用
强烈推荐阅读[官方FAQ](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#overview)和[官方wiki](https://hashcat.net/wiki/)，里面有着十分详细的教学。以下内容全部可以在官方FAQ和Wiki中找到，我只是一台复读机。
hashcat支持多种编码的解码，比如md5，wpa/wpa2，SHA加密等，具体可在安装后使用--help参数查看。对我来说最重要的作用是可以解wpa加密的数据包。

## 可用参数

一般的命令为

```
hashcat [选项] 待解密文件 [字典/格式串]
```
hashcat提供多种选项，选项和参数都可以通过--help查看，这里仅例举几个接下来要用到的

-D：
指定解码硬件，提供如下参数

参数|意义
---|----
1|仅使用cpu解码
2|仅使用GPU解码
3|使用计算卡(不确定，原文：FPGA,DSP,Co-Procsssor)

-m：
指定待解码文件的类型，后跟编码编号，如wpa/wpa2加密对应2500。甚至支持office，rar，zip等文件的密码破解。具体对应的编号表篇幅所限实在是放不下，还请自行查阅官方文档。

-a：
指定攻击模式（Attack-mode），参数如下

参数|意义
--|--
0|使用字典攻击
1|使用字典中的每项依次进行两两组合攻击
3|穷举格式串中每个字符暴力攻击
6|混合（hybrid）攻击
7|混合（hybrid）攻击，顺序和6相反

## 其他特性

- 可以在运行的任何阶段暂停，之后可以轻易的恢复
- 可以通过格式化字符串生成常用的组合，称之为“mask attack”。知道这功能以后省了我一堆的硬盘空间
- 可以随时查看解码进度，另外还有当显卡过热时的自动保护
- 在仅用cpu的情况下的计算性能依然高于aircrack-ng套件，可以推测有着十分优秀的优化。

# 总结

我用aircrack-ng跑了一晚上的密码用这玩意40分钟就能跑开……我为什么没有早点发现这个神器……