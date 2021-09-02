# CTF | 那些比较好玩的stego（正传） - 知乎
自从开了专栏之后（其实是我觉得在专栏上写东西方便），尤其是在写完[读书 | The myths of security](https://zhuanlan.zhihu.com/p/23003966) 之后，居然还收到了 8 个赞，我瞬间怀疑起贵乎的正确使用方式（233，但是还是很开心的）

在这儿给大家续 CTF 系列的正传。当时因为这篇文章，我获得了很多东西：一个乌云账号，意外的稿酬，简历上终于可以多了可以写的东西，等等。然而，6 月份的时候，一直到现在，哎哟我去这篇文章居然被各路大神转载了，我当时的反应如下

![](https://pic2.zhimg.com/80/v2-de33852a9b0e2927d94bb8eb5532b5bd_720w.jpg)
不扯淡了，都过去了，现在本系列已经委托朋友

[@ming Luo](//www.zhihu.com/people/73320d394b4173309359ce551248b171)

进行文章转载的维护。

**所以，各位老司机们，在转载前麻烦和本人打声招呼？OK？**

![](https://pic4.zhimg.com/80/v2-21449496dc88a7a496eda870b7914883_720w.jpg)

好，正文（其实修改过了）开始：

\-------------------------------------------------------------------------------  

## **0X00 引子**

最近国内各种 CTF 层出不穷，看出来国家对于信息安全终于决定下重本了。另外因为本人比较喜欢研究怎么藏东西，所以一开始在 CTF 队组建的时候，就负责了 misc（杂项）中的 Steganography（Stego）部分，中文翻译过来是 “隐写术”。

乌云上也有好几篇关于隐写术的文章，其中以 AppLeU0 大大的那篇《[隐写术总结](https://link.zhihu.com/?target=http%3A//bobao.360.cn/learning/detail/243.html)》最为经典，还有 Gump 大大的《数据隐藏技术》。在这儿，本菜鸟也想和大家分享在 CTF 中见到的比较好玩的 stego 题目。写的不好，请大大们多多包涵。

（啊，某云的文章自己找镜像去吧……）

## **0X01 什么叫隐写术？**

![](https://pic4.zhimg.com/80/v2-543b8699ed9f71fee2443ac963e87db7_720w.png)
题图：无意中被 “隐身墨水” 给隐藏了下半身的 Jerry 正在尝试验证自己的手能不能被 “隐身墨水” 给隐藏（设置变量验证）  

隐写术，在我们的生活中既熟悉而又陌生。记得童年回忆《猫和老鼠》有关于 “隐写术” 的这一集：Jerry 有一天在玩耍的时候，不小心往自己的身上涂了一种“神奇墨水”，然后把自己藏了起来，还成功的把 Tom 戏弄一通并赶走了 Tom。这种“神奇“的墨水，叫“隐形墨水”，写到纸上之后需要特殊处理才可以看得到对方想要给我们传达的信息。

同样的，在信息安全中，隐写术也是举足轻重的一块领域，从我们最早接触到的图种（copy /b 1.jpg+1.torrent 1.jpg），到有事没事就上个头条的 locky，看个图片就中招的 stegosploit，图片木马 Stegoloader，还有就是三大高危 CVE 之一 CVE-2016-3714（ImageMagick 的，linux 下用于处理图像的一个应用），然后也有 CHM 打造后门，等等。总的来说，就和藏东西似的，能藏就藏，而我们就是要去寻找这种蛛丝马迹。  

（现在[JPEG 2000 可执行任意代码漏洞](https://link.zhihu.com/?target=http%3A//www.freebuf.com/vuls/115916.html)已经加入 Stego 豪华漏洞套餐）

## **0X02 常用工具**

手边刚好放了一本很老的大部头：

![](https://pic1.zhimg.com/80/v2-e8217c6ed3590dbd4edf0b1eb325a334_720w.jpg)

（管校图书馆借的），在第 85 页由暗夜舞者的《机密文件的图片隐藏法》里记载了将 zip 文件和 gif 图片的通过文件流读入 ZIP 文件将其合到一起的小办法。最后的文件要怎么打开呢？把. gif 改为. zip 就好啦\~~

另外，在 CTF 中，老司机们可能更偏向于用 UE/Winhex 一类的 16 进制的编辑器去打开待分析文件，然后通过拼写文件头（嗯。常用的文件头有几类？）然后 blah blah 的解决了。然而，新手们尤其是像我这样的菜鸟们，就比较喜欢用工具先分析一下有什么样的发现，这时候，为什么不考虑使用 binwalk 呢？早年在 freebuf 上也有[binwalk](https://link.zhihu.com/?target=http%3A//www.freebuf.com/sectool/15266.html)的身影。

寒假刷 XCTF 的 OJ 的时候，binwalk 用起来很方便，但是也有不方便的时候。比如说我要想藏个字符啥的，这个也是 binwalk 看不出来的。记得最近的某 CTF 曾经出过这道题：

![](https://pic4.zhimg.com/80/v2-fc389d79ef33f984378cb5b1fd82387b_720w.png)

（到后面我还会再用这张图片来进行分析）

人家最后放了个提示 Stegdetect，那时候我们用 binwalk 分析了很久也没分析出来个所以然。图上的，你总不能一个个链接试吧（反正有听说有师傅们通过这个攒了几部小电影的）。然后用 binwalk 分析的时候，提示有压缩包，然后 foremost 出来之后压缩包里攒着另一个压缩包

请根据以上描述，问：此文件属于什么样的数据结构？ （大雾）  

后来，就有了

[@4ido10n](//www.zhihu.com/people/f3a608b2a03677d3aef0dcbd0c52289c)

的《深入理解 JPEG 图像格式 Jphide 隐写》

> Jphide 隐写过程大致为：先解压压缩 JPEG 图像，得到 DCT 系数；然后对隐藏信息用户给定的密码进行 Blowfish 加密；再利用 Blowfish 算法生成伪随机序列，并据此找到需要改变的 DCT 系数，将其末位变为需要隐藏的信息的值，最后把 DCT 系数重新压回成 JPEG 图片。  

关于图片其他方面的隐写，请参考[隐写术总结 - 安全客](https://link.zhihu.com/?target=http%3A//bobao.360.cn/learning/detail/243.html)

到这儿为止，我们已经提了以下几种工具：

-   Binwalk  
-   Stegdetect  

此外根据刚刚提到的这几篇，图片方面我们还可以尝试这些工具：JPHS（由 Stegdetect 检测出来的，这个是分离工具）、Stegsolve、winhex、Power_exif、Pngcheck 等等，此外也可以用编程、或者寻找图片的结尾符 FFD9 等辅助。

如果出的是其他类型的文件呢？

MP4：ffmpeg

PDF：常用的有 wbStego4open

vmdk：Dsfok-tools

以上，当然如果你觉得用工具解不出来的话，那就…… 看下面几道题目压压惊

## **0x03 几道可以分辨恶趣味的题目**

今年被吊打过的 CTF 里，我们可以发现隐写术不那么单调了，体现了隐写术的特点之 “物所能及不能藏”。不光是常见的 jpg、tiff、png，

![](https://pic3.zhimg.com/80/v2-0003025073b090eccf71b23be459c7e6_720w.png)
如果你有细心算过 0x1E 是啥子玩意的话，你应该也不会直接 foremost 分离了……

这道题的正确打开方式如下：

![](https://pic2.zhimg.com/80/v2-45d001e3eb070ea9bef9fcc4094abb25_720w.png)
这道题的解题思路是 exif，exif 承载了这张照片的一些比如相机型号、照相地点等重要信息。然后，我们判断出来这个是什么样的编码就直接解码就好。

另外，在风云杯 2016 上比较有意思的一道 misc04 是关于加密与解密，同时还涉及到 stego，先是 zip 伪加密破解，然后得到一张图片。尔等菜鸟们用 binwalk 分辨出了文件头分出来四张图片，嗯？然后？其实在赛后处理这张图发现了一个诡异的地方：

![](https://pic2.zhimg.com/80/v2-3855f39998165b2558b75b88150da6e5_720w.png)
诡异就诡异在文件尾的 exif 到底是啥？

然后就没然后了……

P.S. 老司机们看透了一切，早早在这儿打好了埋伏，这一次是使用了 zip 伪加密 + stego+base32，杀了个 misc 的回马枪。当时还是太年轻啊……  

还有这道 png 的题目（用 pngcheck 来分析的）

![](https://pic3.zhimg.com/80/v2-2c952bd66ec1c2da9e120a2fb998a1c6_720w.jpg)
这道题通过块损坏和重排来出的题目，把损坏的块处理一下就变成了下面这样![](https://pic2.zhimg.com/80/v2-08bb8aeb5c980d08659417f6826e51dd_720w.png)
重排一下即可。

Word 的基础题

![](https://pic2.zhimg.com/80/v2-c82646fd9a095ca2cc7ff87375186d99_720w.png)
乌云网友评论：你把整个字体颜色调白了也可以达到隐写术的效果![](https://pic3.zhimg.com/80/v2-fbe6cb9e1d44075f936282da02c4bcea_720w.jpg)
PDF

之前见过的隐写题目都很容易，用 wbStego4open 这个工具都可以解出来，这儿一般是以字符串为主。然而，直到发稿之前刚刚看了一道让我整个都 233 的题解：

> The website at [hack.lu CTF 2016](https://link.zhihu.com/?target=https%3A//cthulhu.fluxfingers.net/challenges/4)  
> allows you to download a pdf file. It contains an attachment named _pdf10000.pdf._
>
> Attachment pdf10000.pdf contains another attachment named  
> pdf9999.pdf; pdf9999.pdf contains pdf9998.pdf and so on...
>
> With this behavior in mind, we can write a script to  
> automate the extraction process. The assumption is that pdf0.pdfmight contain the flag.

本来想尝试一下解这道题，然而

!\[](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1366' height='163'></svg>)
!\[](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='111' height='114'></svg>)
那还是不做了，把那道题的题解放出来：

> After "guessing" (a 30.7 MB pdf file containing only few characters and a small image) that the pdf has something embedded in it I started to use pdfdetach to extract the file:
>
> pdfdetach simplepdf_f8004a3ad0acde31c40267b9856e63fc.pdf -saveall  
>
> A new file was created pdf10000.pdf, then when I ran pdfdetach again and observed that it creates a file pdf9999.pdf and so on. The objective was to get to pdf0.pdf so a quick script solved it:  

```php
<?php 
for ($i=10000; $i >= 0; $i--) {
$j = $i + 1;
	if ($j < 10001)
{
		exec("rm pdf$j.pdf");
	}
	exec("pdfdetach pdf$i.pdf -saveall");
	echo $i."\n";
}
?>

```

> When it get to pdf0.pdf I ran it again  
>
> pdfdetach pdf0.pdf -saveall  
>
> And I got start.pdf, ran it again  
>
> pdfdetach start.pdf -saveall  
>
> but I got a file start.pdf with 0 bytes (and lost the original start.pdf, so be careful and backup it !). Something was wrong. I looked on the pdfdetach arguments and :  
>
> pdfdetach start.pdf -save 1 -o x.pdf  
>
> Opened x.pdf and there was the flag :)

```python
#Environment: Linux
#https://cthulhu.fluxfingers.net/static/chals/simplepdf_f8004a3ad0acde31c40267b9856e63fc.pdf#Extract pdf10000.pdf and use the script

from subprocess import call
ctr = 10000
pdfname = "pdf10000.pdf"
while ctr > 0: 
   call(["pdfdetach", "-saveall", pdfname]) #for more info http://www.dsm.fordham.edu/cgi-bin/man-cgi.pl?topic=pdfdetach&sect=1   ctr -= 1
   pdfname = "pdf" + str(ctr) + ".pdf
```

> \----------------------------------------------------------------------------------------------------  
> From the script above, pdf0.pdf was extracted. It contains  
> start.pdf which contains the flag.
>
> The flag is _**flag{pdf_packing_is_fun}**_

pcap 流量分析：

运气好的话，可以尝试使用 foremost+strings，当然特别高阶的方法还是请教各位老司机们……

WAV：  

哎哟说到这个每次都是满脑子的渡口和魂斗罗……

> 常见的 WAV 文件使用 PCM 无压缩编码，这使 WAV 文件的质量极高，体积也出奇大，对于 PCM WAV，恐怕也只有无损压缩的音频才能和其有相同的质量，平时我们见的什么 mp3,wma（不含 wmalossless）和 wav 的质量都是差很远的！这点可以通过频谱看出，即使 320kbps 的 mp3 和 wav 一比，也要自卑了！  
> Wav 格式支持 MSADPCM、CCITTALaw、CCITT μ Law 和其它压缩算法，支持多种音频位数、采样频率和声道，但其缺点是文件体积较大（一分钟 44kHZ、16bit Stereo 的 WAV 文件约要占用 10MB 左右的硬盘空间），所以不适合长时间记录。

之前看别人的 WP 屯了这么几张图

p.s. 脑洞君你好

突然想起来今年的一件事，当时是刚比完四叶草的 CTF 之后，我们队又去了一次国外的小比赛。比赛当中，也是有一首隐写术的题目，给了[rain.wav](https://link.zhihu.com/?target=https%3A//github.com/HackThisCode/CTF-Writeups/blob/master/2016/SCTF/Rain-or-Shine/rain.wav%3Fraw%3Dtrue) （这儿给了其他队伍提取的源文件），那道题当中按照同样的方法却没办法分析出来个所以然，“你听到这歌里有没有杂音？”，然后我和队友把声音调到最大才听出来这首歌的另外的一些声音，通过判断声音位置最后提取到了 flag。  

这件事告诉了我们：比赛的时候，如果有声音类的题目，不要用 10 块钱耳机，认真脸

MIDI：

!\[](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='224' height='183'></svg>)
从来没做出来过 midi 的 flag 的我……  

AVI：

> AVI 含三部分：文件头、数据块和索引块。  
> 其中数据块包含实际数据流，即图像和声音序列数据。这是文件的主体，也是决定文件容量的主要部分。视频文件的大小等于该文件的数据率乘以该视频播放的时间长度，索引块包括数据块列表和它们在文件中的位置，以提供文件内数据随机存取能力。文件头包括文件的通用信息，定义数据格式，所用的压缩算法等参数。

avi 的编码标准不同导致的文件体积太大，比赛中应该不会放那么大的视频文件供 CTFer 下载吧？但谁又能说的准呢？给大家安利一个 MSU VideoStego，是用于分析 avi 文件的小工具。

EXE：

exe 里藏东西其实还涉及到病毒行为分析，有可能是藏在壳内，也有可能是其他地方，具体情况可以使用 IDA 或者 OD 还有 C32Asm 来进行跟进分析。

此话说出后不久，flag 就没了，看这儿：[CTF 中比较好玩的 stego（续一）](https://zhuanlan.zhihu.com/p/22822272)

MP4：

BCTF2016 出过[catvideo](https://link.zhihu.com/?target=https%3A//pan.baidu.com/s/1eQRokOM) （在这儿也给了源文件供各位黑阔分析）， 然后用了 ffmpeg 大法：

> ```text
> ffmpeg.exe -i catvideo-497570b7e2811eb52dd75bac9839f19d7bca5ef4.mp4 -r 30.0 fr_%4d.bmp
>
> ```

然后就 xor 一下得到了这个（我只是打个比方，这道题我原来的脚本不是这么写的）：  

!\[](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='741' height='460'></svg>)

## **0x04 后记**

这一块可以深挖的东西还有很多，都是特别的有意思，如果隐写术和密码学结合起来，也是一道非常有意思的题目，也有见到和算法结合起来的 "png number"（png 套路蛮多的）。个人感觉如果说是顺着 CTF 这一块往下挖的话，也挺考验选手自身的基础的。

不过现在光考察 stego 的题目也已经很少了，反正和这篇文章并不是同一个 level……

\-------------------THE END-------------------

旧文，原文发布于 2016 年 05 月 20 日的乌云知识库。

**在此再次谢谢乌云知识库，谢谢。**   

来源：

[CTFtime.org / Hack.lu CTF 2016 / simplepdf](https://link.zhihu.com/?target=https%3A//ctftime.org/task/2966)

[wav 文件格式分析](https://link.zhihu.com/?target=http%3A//www.cnblogs.com/tiandsp/archive/2012/10/17/2728585.html)

[Rain or Shine (35 pts)](https://link.zhihu.com/?target=https%3A//sardinachanx.gitbooks.io/sctf-2016q1-write-ups/content/rain_or_shine_35_pts.html)

[BCTF - Forensics](https://link.zhihu.com/?target=http%3A//err0r-451.ru/2016-bctf-forensic-catvideo-150-pts/) 
 [https://zhuanlan.zhihu.com/p/23127122](https://zhuanlan.zhihu.com/p/23127122)
