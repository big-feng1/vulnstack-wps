linux dev服务器

目标:

<font style="color:rgb(33, 37, 41);">黑客的IP地址192.168.75.129</font>

<font style="color:rgb(33, 37, 41);">遗留下的三个flag</font>

<font style="color:rgb(33, 37, 41);"></font>

<font style="color:rgb(33, 37, 41);">看有没有web服务</font>

```plain
ss -tlnp
ps aux | grep -E 'nginx|httpd|apache|tomcat|php|java'
```

没有web服务

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789968117770-74f5232f-69f0-41b9-add6-433ddc48d651.png)

查看系统登录日志

```plain
last -ai
```

找到一条远程root登录,ip为192.168.75.129,大概率是攻击者IP

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789968449585-fc877b35-3d44-4861-95b6-676ec473e5f0.png)

查看登录失败/爆破日志

```plain
lastb -ai
```

大量来自192.168.75.129的登录失败日志 确定192.168.75.129为攻击者IP

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789968660345-184c933d-39d2-424f-95b2-bf79a3b8bc10.png)

查看root历史 找到flag{thisismybaby}flag{kfcvme50}

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789970884932-e2658706-d2b9-4aa3-83e4-3f2677b62247.png)

查看系统安全日志

看到攻击者接触过redis

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789971427331-df36c024-8ebc-4cf5-aad6-ae98830ad231.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789971803530-dc0069a6-854d-43c2-96a7-3fe207cf2d6d.png)

在redis.conf找到第三个flag

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789971356683-36835723-1853-4133-8be2-8bce4167c467.png)

