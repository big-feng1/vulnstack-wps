<font style="color:rgb(33, 37, 41);">1.攻击者的IP地址（两个）？</font>`<font style="color:rgb(33, 37, 41);">192.168.126.135</font>``192.168.126.129`

<font style="color:rgb(33, 37, 41);">2.攻击者的webshell文件名？</font>`<font style="color:rgb(33, 37, 41);">system.php</font>`

<font style="color:rgb(33, 37, 41);">3.攻击者的webshell密码？</font>`<font style="color:rgb(33, 37, 41);">hack6618</font>`

<font style="color:rgb(33, 37, 41);">4.攻击者的伪QQ号？</font>`<font style="color:rgb(33, 37, 41);">777888999321</font>`

<font style="color:rgb(33, 37, 41);">5.攻击者的伪服务器IP地址？`256.256.66.88`</font>

<font style="color:rgb(33, 37, 41);">6.攻击者的服务器端口？</font>`<font style="color:rgb(33, 37, 41);">65536 </font>`

<font style="color:rgb(33, 37, 41);">7.攻击者是如何入侵的（选择题）？</font>

<font style="color:rgb(33, 37, 41);">8.攻击者的隐藏用户名？</font>`<font style="color:rgb(33, 37, 41);">hack887$</font>`

<font style="color:rgb(33, 37, 41);"></font>

<font style="color:rgb(33, 37, 41);">D盾查杀,找到已知后门</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789784086704-21080d10-dfa3-4c58-8ae7-54308ad13749.png)

查看webshell代码,密码是`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">hack6618</font>**`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789784261740-ae00a9ce-8a9d-4b7c-a4c7-77291efe6dcb.png)



D盾克隆检测克隆隐藏账号

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789784332535-5fe4413b-6e79-443a-8eb5-f27a423eacfb.png)

查看本地用户和组,确认是hack887$



查看日志apache日志

有个ip一直访问system.php`192.168.126.135`

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789784769513-a044e310-306a-4b4c-a9fc-53b0529a8ddc.png)



翻到腾讯文件夹

C:\Users\Administrator\Documents\Tencent Files\777888999321\ExpressionRecommend

777888999321是qq号?

里面有frp

猜测攻击者用qq上传frp,用frp建立隧道

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789793252886-1cae7512-f6fd-458d-859d-137c49700d5e.png)

查看frp内的配置

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789793319902-a85ff951-d5a3-4305-af7b-3d059c69e9bf.png)

攻击者的伪服务器的IP和端口`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">256.256.66.88</font>**``**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">65536</font>**`



攻击者有隐藏账号,肯定有登录和远程登录行为

<font style="color:rgb(15, 17, 21);">LogonType 为 3（网络登录）或 10（远程桌面登录） 的 4624 事件</font>

<font style="color:rgb(15, 17, 21);">事件查看器中筛选安全日志,保存下来ai解析</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789793003186-d70903b5-9fb9-480c-9972-6a95d6291380.png)

分析结果

> **<font style="color:rgb(15, 17, 21);">LogonType 为 3（网络登录）或 10（远程桌面登录）</font>**<font style="color:rgb(15, 17, 21);"> 的 4624 事件，其 </font>**<font style="color:rgb(15, 17, 21);">源网络地址（IpAddress）字段值均为：</font>**`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.126.129</font>**`<font style="color:rgb(15, 17, 21);">。</font>
>

