# 网络安全管理职业技能大赛WriteUP_F4ke12138的博客-CSDN博客
## 网络安全管理职业技能大赛 WriteUP

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/weixin_39664643/article/details/109073759](https://blog.csdn.net/weixin_39664643/article/details/109073759)

版权

# 网络安全管理职业技能大赛 WriteUP

## 0X01 签到

Base32 解码可得 flag

![](https://img-blog.csdnimg.cn/20201014145551490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

flag{546dcd6b-33cf-408e-a603-27760ada9844}

## 0X02 被黑了\_q1

首先下载压缩文件，解压使用 wireshark 打开，过滤 http 包大致查看

![](https://img-blog.csdnimg.cn/20201014145551825.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

题目要找后台管理员的密码，过滤器里使用  http contains "admin" 过滤，如下图第一个请求包的内容，密码为 admin123，再按照题目中所说操作得到 flag{0192023a7bbd73250516f069df18b500}

![](https://img-blog.csdnimg.cn/20201014145551886.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

## 0X03 被黑了\_q2

接着上题，过滤 http 浏览，大致确定攻击者使用管理员弱口令登陆后台，上传 webshell.php 大马。

此时在 http contains "admin" 过滤器中发现大马执行了 phpinfo 命令

![](https://img-blog.csdnimg.cn/20201014145551850.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

接着右键追踪 http 流，拿到主机名

![](https://img-blog.csdnimg.cn/20201014145551743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

得到 flag{df575d8ac57ee19554a0a87681edb60b}

## 0X04 被黑了\_q3

结合上题 phpinfo 流中得到绝对路径 D:/phpstudy_pro/WWW

![](https://img-blog.csdnimg.cn/20201014145551522.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

如下图中找到上传的大马文件名 webshell.php

![](https://img-blog.csdnimg.cn/20201014145551765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

综上，D:/phpstudy_pro/WWW/webshell.php，得到 flag{8cbb0ea656d8feafadd6b63095a3c0f7}

## 0X05 流量分析

照例过滤 http , 发现进行目录爆破，发现后台登陆口存在 SQL 报错注入，后续进行报错注入

看来题目的意思就是需要找到流量包中 SQL 爆破出来的 flag

过滤器直接 http contains "flag"，一个一个包看过去，前面还有一些假的 flag, 哈哈，直到最后三个包

![](https://img-blog.csdnimg.cn/20201014145551812.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

分三次请求拿到了 flag 的 hex 编码

![](https://img-blog.csdnimg.cn/20201014145551705.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

对其进行 hex 解码即可得到 flag

![](https://img-blog.csdnimg.cn/20201014145551596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

flag{b409f72f-8330-4cd4-9f12-7487e4286694}

## 0x06 XTEA

根据下载附件的密文和 key 知道是 xtea 加密，直接工具解密得到 flag。

![](https://img-blog.csdnimg.cn/20201014145551633.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

## 0x07 神秘邮件：

按照题目的意思，先使用 010-editor 工具查看可直接获得 flag 的前部分和中间部分，

![](https://img-blog.csdnimg.cn/20201014145551526.png)

Base64 解码得到 flag{051d51cc-2528

陆续向上翻阅得到 - 4c65-bf88-

![](https://img-blog.csdnimg.cn/20201014145551704.png)

这是眼睛看瞎也没发现什么了，查找资料发现发现是 zip 缺少头部 504B，添加解压即可得到完整的 docx 文件，在 doc 文档显示背景颜色可发现隐藏的后半部 flag 字样，拼凑可得完整的 flag。

![](https://img-blog.csdnimg.cn/20201014145551704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zOTY2NDY0Mw==,size_16,color_FFFFFF,t_70)

 [https://blog.csdn.net/weixin_39664643/article/details/109073759](https://blog.csdn.net/weixin_39664643/article/details/109073759)
