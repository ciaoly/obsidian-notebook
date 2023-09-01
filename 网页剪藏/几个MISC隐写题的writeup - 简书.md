# 几个MISC隐写题的writeup - 简书
# 几个 MISC 隐写题的 writeup

[爱因斯天](/u/ab421624dc57)关注

0.0662018.01.14 19:10:27 字数 1,184 阅读 4,281

**1.  相机**

![](https://upload-images.jianshu.io/upload_images/10106343-7bc21d901d1eb9c6.png)

因为是简单题，所以直接 winhex 或 notepad++ 打开查找 flag

Flag{iPhone}

**2.  神器**

仔细看看他指的是哪

![](https://upload-images.jianshu.io/upload_images/10106343-5557e70dcd599b15.png)

根据题目提示可用 stegsolve 查看是否隐藏二维码

![](https://upload-images.jianshu.io/upload_images/10106343-4c8372ac2911e0e3.png)

Flag{douniwan}

**3.  好孩子**

好孩子是看不见的

![](https://upload-images.jianshu.io/upload_images/10106343-5be74351fd43a76f.png)

根据题目提示可能隐藏着其他文件，用 binwalk

![](https://upload-images.jianshu.io/upload_images/10106343-b10ead8c724a8ef9.png)

果然不出所料，隐写着一个压缩包

再用 foremost 分离

![](https://upload-images.jianshu.io/upload_images/10106343-41d1483595e0621c.png)

得 Flag{NJY7UbLrKFSDZLCY}

**4.  为什么打不开呢？伤心 QAQ**

[题目所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1pNfw0oN)

解压出的 exe 文件无法打开，接着把解压出的 exe 文件用 binwalk 分析，结果什么都没有，说明此文件可能只是包含了一些数据或缺少头部文件信息，所以用 winhex 打开分析，发现为图片的 base64 的图片编码，在网站解码得到二维码，扫码即得 KEY{dca57f966e4e4e31fd5b15417da63269}

![](https://upload-images.jianshu.io/upload_images/10106343-62c50ed4ed8d01ce.png)

**5.  小苹果**

![](https://upload-images.jianshu.io/upload_images/10106343-f451a3cdd11eca34.png)

老套路先 binwalk, 果然发现有压缩包，foremost 分离出来，结果真的是一首小苹果. MP3，在对此文件 binwalk，并没有什么发现，感觉被刷了。然后再观察图片，扫描二维码得到

\\u7f8a\\u7531\\u5927\\u4e95\\u592b\\u5927\\u4eba\\u738b\\u4e2d\\u5de5

Unicode 解码得  羊由大井夫大人王中工

最终得到 CTF{9158753624}

**6.  号外号外**

[题目 6 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1dGtDIMX)

得到一没有后缀名的文件，binwalk 未发现什么。再用 winhex 打开，发现编码很奇怪，

![](https://upload-images.jianshu.io/upload_images/10106343-6bea0cda24764a24.png)

89504E47 是 GIF 图片格式的头部信息，也就是说它把 ASC 码与十六进制数给颠倒了，于是用 winhex 新建文件，把之前得 ASC 码复制到十六进制的地方，保存得到一张二维码. Gif

![](https://upload-images.jianshu.io/upload_images/10106343-27cc65c545eecc7b.png)

扫描得

号外！号外！山东省大学生信息安全知识大赛开始了！！！！

AAB aaaaa aaa ABB baaa aaa ABBB bbaaa aaa BB bbbbb

一开始认为是培根密码，每五个字母一组，结果前面两个 b 开头根本没有对应的。然后就跟摩斯密码比较像了，把 a 和 b 与. 和 - 相对应，得到 flag{G0ODJOB2OI5}

**7.  要不是 binwalk 差点当 web 了**

[题目 7 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1o9Eug5O)

根据题目提示直接 binwalk，发现有压缩包，foremost 分离压缩包，结果打不开，题目并无暗示密码让你猜的意图，并且题目所给的 web 文件是可以用 wireshark 打开的，打开之，可用 wires hark 自带的工具搜索关键字 pass，不出所料

![](https://upload-images.jianshu.io/upload_images/10106343-85d78a005edab2d2.png)

打开压缩包，得 KEY{ae73e96ab858095eaf1228b1499f8c33}

**8.  压缩包的爆破**

[题目 8 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1smxtbBZ)

![](https://upload-images.jianshu.io/upload_images/10106343-7e1e19e4f786786c.png)

根据注释提示，可知密码为六位纯数字，选择 Advanced Archive Password Recovery 工具进行爆破，打开 word

是一首诗，并没有 flag，把此文件放入 binwalk 分析，发现为 zip 文件，改后缀名之。

在 document.xml 文件中找到

![](https://upload-images.jianshu.io/upload_images/10106343-a4bb94164e3531a9.png)

Flag{**oRuesP57WAaGhBJk**}

**9.   gakki**

先 binwalk 分析，很单纯的一张图片。再观察，她的手指指向上，所以猜测可能要修改图片的高度。

PNG 文件头知识：

（固定）八个字节 89 50 4E 47 0D 0A 1A 0A 为 png 的文件头

\- （固定）四个字节 00 00 00

0D（即为十进制的 13）代表数据块的长度为 13

\- （固定）四个字节 49 48 44

52（即为 ASCII 码的 IHDR）是文件头数据块的标示（IDCH）

\- （可变）13 位数据块（IHDR)

    \- 前四个字节代表该图片的宽

    \- 后四个字节代表该图片的高

    \- 后五个字节依次为：

    Bit depth、ColorType、Compression method、Filter method、Interlace method

\- （可变）剩余四字节为该 png 的 CRC 检验码，由从 IDCH 到 IHDR 的十七位字节进行 crc 计算得到。

改之得

![](https://upload-images.jianshu.io/upload_images/10106343-63e90359bd84b948.png)

**10.  磁盘镜像**

[题目 10 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1qZ8QwR6)

Binwalk 加 foremost 分离

![](https://upload-images.jianshu.io/upload_images/10106343-7f4f0c927bc4471d.png)

**11.  Do you know wireshark?**

[题目 11 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1huiY4fU)

根据目提示，猜测压缩包可能为伪加密。用 winhex 打开，在 ASC 一栏中，找到第二个 PK，即 50 4B 05 06 XX XX XX XX 01 00（XX 为未知数），把 01 改为 00 即可。（改为 02 或 04 也可以，是偶数就行，如果是基数就为加密）。

再用 wires hark 打开，跟踪 TCP 流，找到

flag{d316759c281bf925d600be698a4973d5}

**12.  黑客留下的机密信息**

[题目 12 所用文件](https://link.jianshu.com?t=https%3A%2F%2Fpan.baidu.com%2Fs%2F1dekMuE)

根据题目提示，用 http.request.method=="POST" 过滤数据包，

发现往 shell.php 发送的信息最多，逐个分析

用 base64 解码找到 flag{Inf0rm4ti0n53curity}

若对你有帮助，鄙人荣幸之至。

 [https://wsa.jianshu.io/p/9b233f0497bb](https://wsa.jianshu.io/p/9b233f0497bb)
