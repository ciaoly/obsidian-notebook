
前言
==

CTF-MISC中关于压缩包的隐写题目也是经常出的，今天带给大家的是RAR压缩包的一些谜之操作，希望大家有所收获。

简介
==

RAR(文件头/文件尾:52 61 72 21 1A 07 00)，百度详细解释：

> **RAR**是一种专利文件格式，用于[数据压缩](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%2595%25B0%25E6%258D%25AE%25E5%258E%258B%25E7%25BC%25A9 "https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9")与归档[打包](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%2589%2593%25E5%258C%2585%2F23389687 "https://baike.baidu.com/item/%E6%89%93%E5%8C%85/23389687")，开发者为尤金·罗谢尔（俄语：Евгений Лазаревич Рошал，拉丁转写：Yevgeny Lazarevich Roshal），RAR的全名是“**R**oshal **AR**chive”，即“罗谢尔的归档”之意。首个公开版本RAR 1.3发布于1993年。

**特点**

> RAR通常情况比[ZIP](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2FZIP%2F16684862 "https://baike.baidu.com/item/ZIP/16684862")压缩比高，但压缩/解压缩速度较慢。
> 分卷压缩：压缩后分割为多个文件。
> 固实压缩：把要压缩的视为同一个文件以加大压缩比，代价是取用包中任何文件需解压整个压缩包。
> 恢复记录：加入冗余数据用于修复，在压缩包本身损坏但恢复记录够多时可对损坏压缩包进行恢复。
> [加密](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E5%258A%25A0%25E5%25AF%2586 "https://baike.baidu.com/item/%E5%8A%A0%E5%AF%86")：RAR 2.0使用AES-128-cbc，（rar5.0以后为AES-256CBC）。之前RAR的加密算法为私有。目前均未被直接攻破（至少没有公开），没有密码时只有[暴力破解](https://link.juejin.cn?target=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%259A%25B4%25E5%258A%259B%25E7%25A0%25B4%25E8%25A7%25A3 "https://baike.baidu.com/item/%E6%9A%B4%E5%8A%9B%E7%A0%B4%E8%A7%A3")。
> 缺点：无法有效对付嵌套压缩包,对于密码各异的压缩包WinRAR无法批量处理,对于指定路径中的压缩包不能自动解压,处理大块头压缩包速度缓慢。

伪加密
===

rar4存在伪加密，ra5不存在伪加密

RAR的伪加密与ZIP的伪加密原理相同，造成伪加密的关键都是在一个指定的位标记字段上。

在RAR的第`24`个字节，也就是`010 Editor`显示的文件结构中的`ubyte PASSWORD_ENCRYPTED`字段，修改其字段为`1`即可实现RAR伪加密。

真加密
===

爆破
--

rar4
----

Advanced Archive Password Recovery

有的时候考察爆破，常见的是数字爆破，也会有单纯字母的爆破。

使用工具:Advanced Archive Password Recovery

密码有时候不是通过爆破获得的，出题人可能以其他的形式隐写密码。

rar5
----

hashcat

hashcat的使用

> 下载地址:[hashcat.net](https://link.juejin.cn?target=https%3A%2F%2Fhashcat.net%2F "https://hashcat.net/")

rar2john可以将 RAR 中的密码[哈希](https://link.juejin.cn?target=https%3A%2F%2Fso.csdn.net%2Fso%2Fsearch%3Fq%3D%25E5%2593%2588%25E5%25B8%258C%26spm%3D1001.2101.3001.7020 "https://so.csdn.net/so/search?q=%E5%93%88%E5%B8%8C&spm=1001.2101.3001.7020")提取出来

复制代码

`.\rar2john.exe file` 

有了哈希值， hashcat 暴力破解。如下进行掩码爆破：

dart

复制代码

```
.\hashcat.exe -m 13000 -a 3 '$rar5$16$36fe9da24ec2f10020ba8a989370c697$15$7d2ce8243b92cc889393233fdba54896$8$72203c88592c67e4' ?a?a?a?a
```

### **hashcat参数**

**-a:** 一般使用参数 `-a 3` 即**掩码攻击**

字符集

> ?l: abcdefghijklmnopqrstuvwxyz
>  ?u: ABCDEFGHIJKLMNOPQRSTUVWXYZ
>  ?d: 0123456789
>  ?h: 0123456789abcdef
>  ?H: 0123456789ABCDEF
>  ?s: «space»!"#$%&'()*+,-./:;<=>?@\[\]^_{|}~
>  ?a: ?l?u?d?s
>  ?b: 0x00 - 0xff

`?a?l?l?l?d?d?d?d` 代表第一位为任意字符，第二到第四位为小写字母，后四位为数字

**–increment:** 自增遍历模式（不确定密码长度的情况）

**–increment-max:** 规定自增模式密码最长的长度

**–increment-min:** 规定自增模式密码最短的长度

其他
==

一些ZIP格式中可以破解的方式RAR也可以使用，这里就不过多阐述，有兴趣的小伙伴可以自己去了解。

结尾
==

以上给大家带来的是一些RAR压缩包的一些套路，是不是很有创意，大家学会了也可以自己尝试加密或破解一下，若大家还有知道其他的创意套路也可以在评论区留言互相学习，感谢阅读。