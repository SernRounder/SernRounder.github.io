---
title: 模拟环境下对wpa2加密的WiFi网络破解
date: 2018-12-29 00:09:37
tags:
- WiFi
- 无线安全
- 工具使用
categories: 练手记录
---

前置知识请参看我之前的文章→[HashCat](https://www.sern.site/2018/12/25/HashCat学习笔记/)、[Airacrack-ng](https://www.sern.site/2018/12/27/Aircrack-ng-渗透测试组件学习笔记/)、[Deauth攻击](https://www.sern.site/2018/12/28/取消验证洪水攻击%EF%BC%88Deauth%EF%BC%89/)、[WPA加密原理](https://www.sern.site/2018/12/28/WPA2加密原理/)
# 搭建测试环境
**注意，在未经同意的情况下接入他人网络或获取他人传输的数据是违法行为，请务必在进行渗透测试前取得授权**
<!--more-->
## 靶机
由于没有多余的路由器，目标AP（AccessPoint）用手机热点代替，使用的密码为我以前的常用密码，连接的客户端为另一台电脑。
## 攻击端及工具
系统使用Kali Linux，本来打算使用Ubuntu的，但是似乎aircrack-ng在Ubuntu下运行不是特别稳定，所以换到Kali Linux。同时使用Windows下的HashCat作为破解密码（跑包）的工具。

# 测试流程
为了方便，这里用ssh连接远程Linux来完成
## 测试目标确定
首先在Kali中将网卡调到混杂模式，来监听所有范围内的WiFi数据包。
```
airmon-ng start wlan0
```
![](https://i.loli.net/2018/12/29/5c264904e9b00.png)
之后使用airodump-ng来抓取数据包，本来应该先通过没有参数的模式扫描全部信道逐步确定攻击目标，但由于预先把AP设置在了6信道，就只获取信道6的数据包就可以。
```
airdoump-ng -c 6 -w test wlan0mon
```
可以看到设置的AP被探测到，同时探测到的还有接入AP的客户端。
![](https://i.loli.net/2018/12/29/5c26490e8c5f7.png)

## 捕获wpa加密的四次握手包
由于客户端与AP是一直连接的状态，需要通过Deauth攻击将客户端踢下线，迫使它重新发送握手包。这里我们使用aireplay-ng模块
```
aireplay-ng -0 20 -a 4e:49:e3:78:f2:2f  -c 18:1d:ea:e2:b6:e8 wlan0mon
```
发送20次取消验证帧。
可以看到目标很快就进行了再次连接，握手包也成功抓到。
![](https://i.loli.net/2018/12/29/5c26490d73728.png)

## 进行数据包破解
由于aircrack-ng的破解功能实在是堪忧，这里我们切换到Windows下换HashCat。至于切换到Windows下的原因，HashCat不占cpu，我可以一边打游戏一边等着破解。
字典选用我之前网上找的6个多G的弱密码合集
抓回来的数据包不能直接用hashcat破解，需要用hashcat_utils，一个官方的转换工具转换成.hccapx格式的数据包。
![](https://i.loli.net/2018/12/29/5c264919f2d16.png)
破解完成，密码get。
![](https://i.loli.net/2018/12/29/5c26491228115.png)
至此，WiFi网络已被攻破，在客户端与AP之间传输的数据对于攻击者来说已经垂手可得。
切回Kali中，使用airdecap-ng来解码数据包。
## 数据包分析
![](https://i.loli.net/2018/12/29/5c26490fea93b.png)
可以看到我之前浏览app和网页的数据一览无余。但是由于目前https加密的普及，泄露的也只有部分网站而已。
# 总结
WiFi密码被攻破的危害不止隐私泄露，攻击者接入网络后可以对路由进行攻击，或者使用arp欺骗等手段来伪装为路由。另外如果建立一个与目标AP相同的伪造WiF，再通过Deauth让目标AP离线，那么客户端会自动连接到攻击者的电脑上。通过这些后续攻击手法攻击者可以轻易的的通过中间人攻击来获得客户端发送到网上的密码、用户名等敏感信息，也可以通过窃取cookies来劫持会话。
可以通过提高密码复杂度以及定期更换密码来提高WiFi安全度，另外务必关闭路由器的pin功能，之后会继续讲如果通过pin码来破解WiFi密码，那就是另一个博文了。~~我的肝有点疼~~
