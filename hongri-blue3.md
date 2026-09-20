1. <font style="color:rgb(33, 37, 41);">攻击者的两个IP地址</font>`<font style="color:rgb(33, 37, 41);">192.168.75.130</font>``192.168.75.129`
2. <font style="color:rgb(33, 37, 41);">隐藏用户名称</font>`<font style="color:rgb(33, 37, 41);">hack6618$</font>`
3. <font style="color:rgb(33, 37, 41);">黑客遗留下的flag【3个】</font>



d盾查杀扫描phpstudy目录

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789903367174-61beb9fc-f35b-463e-94a3-009afb35dba2.png)

找到两个webshell

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789903394890-fe0896f2-137d-421c-914c-e2298fcec449.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789903439001-36e2eb92-0de1-480f-9abb-5ef0e13ec388.png)



本地用户和组找到攻击者隐藏账号

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789903547462-a15dbefb-36e0-4a10-8efa-c47178ba1692.png)

筛选日志事件伪4624,导出

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789903794319-d979597f-5f55-45c2-9b04-c8527e0d105c.png)

找到登录类型为10的日志

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789904043383-281903ca-a2c3-4b95-8a14-bb06734c3307.png)

目标用户名是hack6618$,对上了,ip地址为192.168.75.130,攻击者的第一个IP





读日志D:\phpstudy_pro\Extensions\Apache2.4.39\logs

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789904666337-d23d49a4-f276-4adf-a94a-3542d38dc094.png)

发现爆破,IP地址为192.168.75.129

<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">taskschd.msc找计划任务</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789905058519-b5ffa8e0-64e8-458e-a2de-2994c9d534fe.png)

找到第一个flag`flag{zgsfsys@sec}`

找到这个计划任务的执行脚本

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789905725986-d8ee8f10-49ae-47fd-a075-13e3d7ae7c03.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789905775363-14740b35-489e-4e27-ba48-5104d31ee53a.png)第二个flag flag{888666abc}

攻击者攻击肯定要经过web入口

访问本地服务,有admin登录页面[http://127.0.0.1/zb_system/login.php](http://127.0.0.1/zb_system/login.php)

不知道密码,直接看本地代码D:\phpstudy_pro\WWW\zb_system\login.php

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789908268025-b8720120-7332-45df-ae87-cded76c36e36.png)

文件在./function/c_system_base.php

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789908559797-d8a4aa33-ec28-40c6-9459-5b3ec1860cdd.png)

<font style="color:rgb(15, 17, 21);">数据库配置保存在网站根目录下的 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">zb_users/c_option.php</font>`<font style="color:rgb(15, 17, 21);"> 文件里。</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789908623092-11154b94-d0c8-4910-b97a-b900c993ffb8.png)

拿到数据库配置信息

+ **<font style="color:rgb(15, 17, 21);">数据库类型</font>**<font style="color:rgb(15, 17, 21);">：mysqli</font>
+ **<font style="color:rgb(15, 17, 21);">主机</font>**<font style="color:rgb(15, 17, 21);">：localhost</font>
+ **<font style="color:rgb(15, 17, 21);">用户名</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">zblog123</font>`
+ **<font style="color:rgb(15, 17, 21);">密码</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">zblog123</font>`
+ **<font style="color:rgb(15, 17, 21);">数据库名</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">zblog123</font>`
+ **<font style="color:rgb(15, 17, 21);">表前缀</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">zbp_</font>`

登录数据库

在这个位置打开终端

```plain
mysql -u zblog123 -pzblog123 zblog123
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789908896530-4756968f-a414-4643-bfd6-bb41ae0d4328.png)

flag就在数据库里

 flag{H@Ck@sec}

