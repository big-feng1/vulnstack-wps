<font style="color:rgb(33, 37, 41);">1.攻击者的shell密码</font>`<font style="color:rgb(33, 37, 41);">rebeyond</font>`

<font style="color:rgb(33, 37, 41);">2.攻击者的IP地址</font>`<font style="color:rgb(33, 37, 41);">192.168.126.1</font>`

<font style="color:rgb(33, 37, 41);">3.攻击者的隐藏账户名称</font>`<font style="color:rgb(33, 37, 41);">hack168$</font>`

<font style="color:rgb(33, 37, 41);">4.攻击者挖矿程序的矿池域名(仅域名)</font>`<font style="color:rgb(33, 37, 41);">http://wakuang.zhigongshanfang.top</font>`

<font style="color:rgb(33, 37, 41);">5.有实力的可以尝试着修复漏洞</font>

# <font style="color:rgb(33, 37, 41);">攻击者的shell密码</font>
D盾查杀选择phpstudy目录

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789734336120-dc277304-11ff-4c01-a1af-552278ba1541.png)

找到webshell

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789734394906-3d55a5dd-5c3f-46ce-a6bf-fb14f6c5e947.png)

# <font style="color:rgb(33, 37, 41);">攻击者的IP地址</font>
查找apache日志

phpstudy_pro/Extensions/Apache2.4.39/logs/

在accesslog中找到shell.php,定位出攻击者ip`192.168.126.1 `

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789734598519-582c639c-7df8-4f97-8a4d-7d1b9685e320.png)

# <font style="color:rgb(33, 37, 41);">攻击者的隐藏账户名称</font>
**<font style="color:rgb(31, 35, 40);">查看服务器是否存在可疑账号、新增账号</font>**

<font style="color:rgb(31, 35, 40);">查看本地用户和组</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789727964807-58fa11b3-a255-4833-be67-1be2eccb0c04.png)

在Administrator用户组找到可疑账号

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789728675489-10e929a0-de1f-42de-bda4-f391dc4ded7a.png)

禁用账户

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789728399674-22654404-af22-4789-95a3-284586e920ee.png)

**<font style="color:rgb(31, 35, 40);">查看服务器是否存在隐藏账号、克隆账号</font>**

<font style="color:rgb(31, 35, 40);">使用D盾,检测到的隐藏账号就是刚刚警用的</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789729247028-cff4f66d-16ba-4ed4-95ac-a661c8d29e54.png)

<font style="color:rgb(31, 35, 40);"></font>

**<font style="color:rgb(31, 35, 40);">结合日志，查看管理员登录时间、用户名是否存在异常</font>**

<font style="color:rgb(31, 35, 40);">ctrl+r打开事件查看器eventvwr.msc</font>

<font style="color:rgb(31, 35, 40);">筛选事件4720:新建用户账号</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789729879639-810b2188-03f9-4ab8-abfa-8c64ba8fbaa3.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789729915713-d7d56acb-5ff7-406d-9446-d399b3ee6695.png)

# <font style="color:rgb(33, 37, 41);">攻击者挖矿程序的矿池域名</font>
在攻击者隐藏账号下找到exe

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789735057905-8b8466b0-e628-4041-aba2-503e61503afd.png)

**<font style="color:rgb(15, 17, 21);">第一步：解包 exe</font>**

<font style="color:rgb(15, 17, 21);">访问在线解包网站：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">https://pyinstxtractor-web.netlify.app/</font>`

<font style="color:rgb(15, 17, 21);">把 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Kuang.exe</font>`<font style="color:rgb(15, 17, 21);"> 上传上去</font>

**<font style="color:rgb(15, 17, 21);">第二步：反编译 pyc</font>**

<font style="color:rgb(15, 17, 21);">访问 PyLingual：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">https://pylingual.io/</font>`

<font style="color:rgb(15, 17, 21);">把 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Kuang.pyc</font>`<font style="color:rgb(15, 17, 21);"> 拖进去，它会自动反编译成可读的 Python 源代码</font>

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/47774866/1789735503517-d1a5d648-4f7c-4262-9947-962c50167df2.png)

