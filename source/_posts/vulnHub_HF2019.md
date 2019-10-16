---
title: VulnHub_Hacker Fest 2019
date: 2019-10-14 19:57:11
tags:
- VulnHub
- 渗透测试
categories: WriteUP
---
# 靶机环境

官网链接：[https://www.vulnhub.com/entry/hacker-fest-2019,378/](https://www.vulnhub.com/entry/hacker-fest-2019,378/)

靶机下载：[https://patron-it.cz/uploads/HF2019-Linux.ova](https://patron-it.cz/uploads/HF2019-Linux.ova)

<!--more-->
# 信息搜集
masscan扫描内网网段，发现192.168.56.115上线

![](https://i.loli.net/2019/10/15/AhxPwUDBeJcjKLq.png)

切换nmap，-A自动挡深入扫描

```nmap
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-15 20:15 CST
Nmap scan report for 192.168.56.115
Host is up (0.00036s latency).
Not shown: 996 filtered ports
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-rw-r--    1 ftp      ftp           420 Nov 30  2017 index.php
| -rw-rw-r--    1 ftp      ftp         19935 Sep 05 08:02 license.txt
| -rw-rw-r--    1 ftp      ftp          7447 Sep 05 08:02 readme.html
| -rw-rw-r--    1 ftp      ftp          6919 Jan 12  2019 wp-activate.php
| drwxrwxr-x    9 ftp      ftp          4096 Sep 05 08:00 wp-admin
| -rw-rw-r--    1 ftp      ftp           369 Nov 30  2017 wp-blog-header.php
| -rw-rw-r--    1 ftp      ftp          2283 Jan 21  2019 wp-comments-post.php
| -rw-rw-r--    1 ftp      ftp          3255 Sep 27 13:17 wp-config.php
| drwxrwxr-x    8 ftp      ftp          4096 Sep 29 07:36 wp-content
| -rw-rw-r--    1 ftp      ftp          3847 Jan 09  2019 wp-cron.php
| drwxrwxr-x   20 ftp      ftp         12288 Sep 05 08:03 wp-includes
| -rw-rw-r--    1 ftp      ftp          2502 Jan 16  2019 wp-links-opml.php
| -rw-rw-r--    1 ftp      ftp          3306 Nov 30  2017 wp-load.php
| -rw-rw-r--    1 ftp      ftp         39551 Jun 10 13:34 wp-login.php
| -rw-rw-r--    1 ftp      ftp          8403 Nov 30  2017 wp-mail.php
| -rw-rw-r--    1 ftp      ftp         18962 Mar 28  2019 wp-settings.php
| -rw-rw-r--    1 ftp      ftp         31085 Jan 16  2019 wp-signup.php
| -rw-rw-r--    1 ftp      ftp          4764 Nov 30  2017 wp-trackback.php
|_-rw-rw-r--    1 ftp      ftp          3068 Aug 17  2018 xmlrpc.php
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.56.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open  ssh      OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey:
|   2048 b7:2e:8f:cb:12:e4:e8:cd:93:1e:73:0f:51:ce:48:6c (RSA)
|   256 70:f4:44:eb:a8:55:54:38:2d:6d:75:89:bb:ec:7e:e7 (ECDSA)
|_  256 7c:0e:ab:fe:53:7e:87:22:f8:5a:df:c9:da:7f:90:79 (ED25519)
80/tcp    open  http     Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 5.2.3
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Tata intranet &#8211; Just another WordPress site
10000/tcp open  ssl/http MiniServ 1.890 (Webmin httpd)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Login to Webmin
| ssl-cert: Subject: commonName=*/organizationName=Webmin Webserver on Linux-Debian
| Not valid before: 2019-09-09T13:32:42
|_Not valid after:  2024-09-07T13:32:42
|_ssl-date: TLS randomness does not represent time
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: bridge|general purpose
Running (JUST GUESSING): Oracle Virtualbox (98%), QEMU (93%)
OS CPE: cpe:/o:oracle:virtualbox cpe:/a:qemu:qemu
Aggressive OS guesses: Oracle Virtualbox (98%), QEMU user mode network gateway (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT     ADDRESS
1   0.08 ms 10.0.2.2
2   0.11 ms 192.168.56.115

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 51.31 seconds
```
## 目标结构

- 21
    - FTP
- 22
    - ssh
- 80
    - word press
- 10000
    - webmin
# 攻击面
## 21端口的FTP
从之前nmap的扫描结果可以看到，目标的FTP允许匿名登录，而且ftp目录同时也是word press的根目录，从而出现了网站源码和网站结构泄露。拖去D盾里扫了一遍，没有给我报告什么高危的命令执行的利用点，不过等级低的倒是有两个，在wp-google-maps。接下来要是其他路线走不通的话可以回来源代码审计。
```
Connected to 192.168.56.115.
220 (vsFTPd 3.0.3)
Name (192.168.56.115:root): Anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> passive
Passive mode on.
ftp> ls
227 Entering Passive Mode (192,168,56,115,185,115)
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           420 Nov 30  2017 index.php
-rw-rw-r--    1 ftp      ftp         19935 Sep 05 08:02 license.txt
-rw-rw-r--    1 ftp      ftp          7447 Sep 05 08:02 readme.html
-rw-rw-r--    1 ftp      ftp          6919 Jan 12  2019 wp-activate.php
drwxrwxr-x    9 ftp      ftp          4096 Sep 05 08:00 wp-admin
-rw-rw-r--    1 ftp      ftp           369 Nov 30  2017 wp-blog-header.php
-rw-rw-r--    1 ftp      ftp          2283 Jan 21  2019 wp-comments-post.php
-rw-rw-r--    1 ftp      ftp          3255 Sep 27 13:17 wp-config.php
drwxrwxr-x    8 ftp      ftp          4096 Sep 29 07:36 wp-content
-rw-rw-r--    1 ftp      ftp          3847 Jan 09  2019 wp-cron.php
drwxrwxr-x   20 ftp      ftp         12288 Sep 05 08:03 wp-includes
-rw-rw-r--    1 ftp      ftp          2502 Jan 16  2019 wp-links-opml.php
-rw-rw-r--    1 ftp      ftp          3306 Nov 30  2017 wp-load.php
-rw-rw-r--    1 ftp      ftp         39551 Jun 10 13:34 wp-login.php
-rw-rw-r--    1 ftp      ftp          8403 Nov 30  2017 wp-mail.php
-rw-rw-r--    1 ftp      ftp         18962 Mar 28  2019 wp-settings.php
-rw-rw-r--    1 ftp      ftp         31085 Jan 16  2019 wp-signup.php
-rw-rw-r--    1 ftp      ftp          4764 Nov 30  2017 wp-trackback.php
-rw-rw-r--    1 ftp      ftp          3068 Aug 17  2018 xmlrpc.php
226 Directory send OK.
```

![](https://i.loli.net/2019/10/15/ckmHbInylEVzx5j.png)
## 80端口的服务
由于目标由wordpress搭建，可以考虑利用wpscan扫描器进行测试。

![](https://i.loli.net/2019/10/15/19znTPucBSwGU7l.png)

扫描看到其中一个插件的版本过低。同时之前的D盾扫描结果中也报告了这个插件可能有问题，后面重点关注。

## 10000端口的webmin
测试利用最近的webmin后门，失败。

![](https://i.loli.net/2019/10/15/6GkrBaJpmTRHwZ8.png)

# 攻击流程

考虑从80端口开始渗透。
从metasploit的漏洞库中找到了word press过期插件对应的漏洞和exp，开始注入。
![](https://i.loli.net/2019/10/15/2YIek85LhKl96dF.png)
设置下参数，exploit后就可以看到爆出来的用户名和密码。
![](https://i.loli.net/2019/10/15/KQH9SJxTNmGb7r3.png)
放到hashcat里跑一下字典就能得到对应的明文，后面的webmaster@none.local是用户名
![](https://i.loli.net/2019/10/15/8fs1pFKilVPS3YD.png)
得到用户名密码：
```
user: webmaster@none.local
passwd: kittykat1
```
至此已经可以登录word press后台了。

但是我们先缓一缓，别忘了还有个webmin，尝试使用当前的用户名密码组合登录。
![](https://i.loli.net/2019/10/15/pB6jLz53Y1EK7ht.png)
幸福有点突然，再尝试下ssh
![](https://i.loli.net/2019/10/15/Mf9EgGDS8JO2oB4.png)
真就一个密码走天下
当前用户在sudoer里面，直接```sudo su```提权，变成root
![](https://i.loli.net/2019/10/15/peX5G7LwV3ycojC.png)

# 收获

- 熟悉一遍ftp命令的使用
- 发现了wpscan这个神器
- 以后自己的系统不同部分之间的密码务必差异化。