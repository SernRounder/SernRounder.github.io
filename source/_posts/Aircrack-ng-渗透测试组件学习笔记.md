---
title: Aircrack-ng 渗透测试组件学习笔记
date: 2018-12-27 17:19:11
tags:
- WiFi
- 无线安全
- 工具使用
categories: 学习笔记
---
# 摘要
Aircrack-ng是一套完整的，针对WiFi网络安全的渗透测试工具。它拥有多种功能
- 侦听：嗅探并储存WiFi数据包以供将来的第三方工具处理
- 攻击：重放攻击，解除认证攻击，WiFi钓鱼和注入攻击等
- 测试：测试WiFi基站的可靠性和稳定性
- 破解：支持WEP和WPA/WPA2破解
<!--more-->
惯例，完全版请移步[官网](http://www.aircrack-ng.org/doku.php),以下内容是我的学习记录，出现偏差请以官方为准。

# 套件组成

- airbase-ng
- aircrack-ng
- airdecap-ng
- airdecloak-ng
- airdriver-ng - REMOVED in 1.2 rc 1
- airdrop-ng
- aireplay-ng
- airgraph-ng
- airmon-ng
- airodump-ng
- airolib-ng
- airserv-ng
- airtun-ng
- besside-ng
- dcrack
- easside-ng
- packetforge-ng
- tkiptun-ng
- wesside-ng

接下来仅就常用组件进行介绍

## airedcap-ng
这是一个用于解码wep/wpa/wpa2加密的数据包。也可以用来移除未加密的WiFi数据包的无线报头。它会输出一个后缀为-dec.cap的新文件。
一般使用形式为
```
airdecap-ng [options] pcapfile
```
支持的选项如下

选项|描述
:-:|:-:
-l|不移除802.11报头
-b|指定接入点的MAC地址
-k|指定16进制的PMK
-e|目标WiFi名称
-p|目标的wpa2/wpa密码
-w|目标的wep密码（16进制形式）
[参考网站](http://www.aircrack-ng.org/doku.php?id=airdecap-ng)

## aircrack-ng
这是一个802.11 WEP和WPA/WPA2-PSK密钥破解程序。
它能够通过字典破解出WPA/WPA2加密的密钥。它不一定需要wpa加密过程中的四次握手包。实际上它只需要第二次和第三次或者第三、第四次握手包就能够开始解码。
示例：
```
aircrack-ng [options] CaptureFile(s)
```
[参考网站](http://www.aircrack-ng.org/doku.php?id=aircrack-ng)

## airoudmp-ng
该模块用于抓取原始的WiFi数据包。它也可以在有可用的GPS模块的情况下绘制周围的WiFi位置地图，把热点位置标记在地图上。
在使用该模块前，需要使用airmon-ng把网卡置于混杂模式实现监听。
具体参数可使用--help查看
[参考网站](http://www.aircrack-ng.org/doku.php?id=airodump-ng)
样例：
使用网卡wlan0抓取信道6上所有WiFi的数据包，保存项目名称为test
```
airodump-ng -c 6 -w test wlan0
```
使用wlan0抓取信道6上bssid为00：00：00：00：00：00的基站的数据包
```
airodump-ng -c 6 --bssid 00:00:00:00:00:00 wlan0
```
用wlan0抓取范围内所有的数据包
```
airdoump-ng wlan0
```
网卡是必须指定的，其他为可选参数。
使用界面类似于：
![]()
## airmon-ng
该模块用于将网卡调至混杂模式
示例：
```
airmon-ng start|stop wlan0
```
一般情况下网卡会被重命名为wlan0mon

## aireplay-ng
用于注入数据包。
主要用于辅助airdoump-ng捕获数据包。它可以使用取消验证洪水攻击来阻断目标与路由的连接，这样目标重新连接网络时我们就能获得他的握手包来进行后续破解。
[参考](http://www.aircrack-ng.org/doku.php?id=aireplay-ng)