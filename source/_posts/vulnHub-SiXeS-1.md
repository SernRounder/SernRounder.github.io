---
title: vulnHub_SiXeS_1
date: 2019-10-20 18:40:53
tags: 
- VulnHub
- 渗透测试
categories: WriteUp
---
# 靶机环境
## 暂未取得所有flag，进度（4/6）
官网链接：https://www.vulnhub.com/entry/sixes-1,380/
下载地址：https://download.vulnhub.com/sixes/SiXeS-1aa67eae208f9fcc3785c1e622805a35.ova
<!--more-->
# 信息搜集
目标在192.168.56.116上线
![](https://i.loli.net/2019/10/20/4cIo9bnPeLUYQKS.png)
nmap走一轮
```
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-20 19:02 CST
mass_dns: warning: Unable to determine any DNS servers. Reverse DNS is disabled. Try using --system-dns or specify valid servers with --dns-servers
Nmap scan report for 192.168.56.116
Host is up (0.00013s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-r--r--r--    1 0        0             233 Oct 03 20:44 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.56.101
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b2:b1:45:a2:01:2b:10:7e:01:04:c3:5d:4d:43:5d:4d (RSA)
|   256 c2:c3:e2:fa:37:97:b7:32:1a:9d:43:96:f1:72:01:e3 (ECDSA)
|_  256 a9:87:18:d6:a3:a1:ad:7e:85:d0:3f:cf:d7:bd:49:c9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: SiXeS - Making Team Fortress 2 great again
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
目标开放了21，22，80端口，并且21端口的ftp还是存在匿名登录。
# 进行渗透
## ftp
匿名登录进去get一下note，第一个flag到手
![](https://i.loli.net/2019/10/20/ChMQP8IHmkJrSdR.png)
## 80端口
一个web服务
![](https://i.loli.net/2019/10/20/z9Frk1VsKlJ8Eb7.png)
主页上有一个管理员登录的入口，需要用户名密码。
还有contact.php页面可以给网站留言
![](https://i.loli.net/2019/10/20/Z3CvLHatbXRxjFV.png)
用nc打开本机的某个端口接受连接，同时把自己的ip和端口号交上去。远端会用url开头的协议与你通信。
![](https://i.loli.net/2019/10/20/xVwXn1JsD9Eg3Ta.png)
收到通信流量后会发现cookie也带过来了。
下一步修改cookie，访问admin.php就可以看到管理后台了。同时页面也给出了第二个flag
![](https://i.loli.net/2019/10/20/mZjsuVI7Fbg6qly.png)
## 文件上传
只允许上传图片格式的文件。但是并不是对文件后缀进行检查，而是直接看文件头。随便找一个图片文件，在末尾追加一句话木马，改名成a.php上传。
上传位置可以通过复制网站中其他图片的位置得到，在/img/目录下。
![](https://i.loli.net/2019/10/20/LEvdO1RrU95aNyz.png)
之后写入反弹shell，传入meterpreter生成的shell木马就可以获得一个meterpreter环境了。（msf真的香）
![](https://i.loli.net/2019/10/20/UxAklcYSZCzwiEv.png)
之后进入/home文件夹，拿到第三个flag
![](https://i.loli.net/2019/10/20/lVgDGufXq7L8oSd.png)
在config.php里面有sql数据库的明文密码和用户，使用这个登录数据库。
![](https://i.loli.net/2019/10/20/BYnfC25D1qXtwMH.png)
之前生成的shell无法交互式的输入密码，这里再用python生成一个。由于远端的python需要用python3来调用，需要一点小小的修改。
```
python3 -c "import pty;pty.spawn('/bin/bash')"
mysql -u root -p
```
之后得到数据库
![](https://i.loli.net/2019/10/20/ITnk7uVBYsgrCPZ.png)
使用sixes数据库，里面放着第四个flag
![](https://i.loli.net/2019/10/20/ZL7eA49bRVCPcDn.png)
# 总结
说实话，我感觉我走的有点偏，这个似乎是想让你用sql注入的方式搞出来的，而不是拿到webshell后登录查询搞出来。
后面我还拿到了webmaster的密码对应的md5值，可惜跑遍了字典都没能还原出密码。暂时卡在这里了。对于提权还没有什么思路。也试了几个cve，可惜都跑不动。常见的sudo错配问题也试了，没有用。
![](https://i.loli.net/2019/10/20/jSB4FxRyJGVhWrH.png)
