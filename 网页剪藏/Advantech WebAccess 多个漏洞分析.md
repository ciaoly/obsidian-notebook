# Advantech WebAccess 多个漏洞分析
# Advantech WebAccess 多个漏洞分析

2019 年 08 月 20 日 2019 年 08 月 20 日  
[漏洞分析](/category/vul-analysis/)

## 目录

-   [Advantech WebAccess 历史漏洞统计情况](#advantech-webaccess)
-   [Advantech WebAccess Node 多个 ZDI 漏洞分析](#advantech-webaccess-nodezdi)
    -   [1、CVE-2017-16720 命令执行漏洞](#1cve-2017-16720)
    -   [2、CVE-2019-10993 指针解引用漏洞](#2cve-2019-10993)
-   [Advantech WebAccess Node 漏洞挖掘](#advantech-webaccess-node)
    -   [1、CNVD-2019-23511 任意文件删除漏洞（中危）](#1cnvd-2019-23511)
    -   [2、CNVD-2019-23512 命令执行漏洞（高危）](#2cnvd-2019-23512)
    -   [3、CNVD-2019-23513 命令执行漏洞（高危）](#3cnvd-2019-23513)
-   [结 语](#_1)

**作者：启明星辰 ADLab**  
**文章来源：[Advantech WebAccess 多个漏洞分析](https://mp.weixin.qq.com/s?__biz=MzAwNTI1NDI3MQ==&mid=2649614384&idx=1&sn=c7eaaff26938d4dc92ef32cd851a1e2d&chksm=83063520b471bc36368de15bf0f195de8bffc6fbb4a6c4cb3648ad4b213872e0d50c2bc5e508&mpshare=1&scene=1&srcid=0820HxxoMQBWllPfsWNVO438&sharer_sharetime=1566286107538&sharer_shareid=bafb2678ed1f77a340809d0b35c3d277&key=7ee95d8bb7f7df5e0ccef7fc7d673498c302f1bdab2a86012a7d8089b909ae048f03157897237ed81df7815c273a5d06c2264855be26ba29e24dfacc861b80fe16153913f8a97555cc072752458ea482&ascene=1&uin=MzM5ODEzOTY3MQ%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=eT1pa5S9ibxZcgeIuAzHzpptuOANeJIpacaA65vpHM2sqJemoxAAfF9pdrsRUzQI "外出")**

Advantech WebAccess 是研华科技开发的完全基于 IE 浏览器的 HMI/SCADA 监控软件，其最大特点就是全部的工程项目、数据库设置、画面制作和软件管理都可以通过使用标准的浏览器完成，不仅可以实现系统的远程控制而且还能进行工程的开发和维护。近日 ZDI 公布了多个 WebAccess 的漏洞预警 (CVE-2019-10985、CVE-2019-10993、CVE-2019-1099、 CVE-2019-6550 以及 ZDI-19-584 ~ ZDI-19-623、ZDI-19-691)，其中包括多个内存破坏漏洞（ZDI-19-595~ ZDI-19-614）以及栈溢出漏洞（ZDI-19-585、ZDI-19-327~ZDI-19-330、ZDI-19-325、ZDI-19-323、ZDI-19-592、ZDI-19-594、ZDI-19-589、ZDI-19-588、ZDI-19-586)，部分内存破坏漏洞可以在受影响的系统中执行任意代码，但是大部分内存破坏漏洞利用条件较为苛刻。同时，由于 Advantech WebAccess 许多模块并没有开启 ASLR、DEP 等系统相关安全机制，使得栈溢出等漏洞在受影响的系统中容易造成代码执行。

### Advantech WebAccess 历史漏洞统计情况

通过追踪 CVE 漏洞数据库中的 Advantech WebAccess 历史漏洞，启明星辰 ADLab 发现从 2011 年到 2019 年合计有 134 个漏洞被披露。图 1 是我们统计的每一年的漏洞披露数量：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/13adbd4a-e165-4ec9-9a6b-c75514b63c87.jpg-w331s)

图 1 Advantech WebAccess 历年 CVE 漏洞数量情况

从图中可以看出，其漏洞数量总体上随着年份上下波动。2014 年出现一次井喷达到 26 个，而 2015 年漏洞披露只有 5 个，随后出现逐年上升趋势。我们对 WebAccess 的漏洞类型信息进行了梳理，如图 2：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/08f56076-076e-4f4e-96c7-48d07b3ce7d0.jpg-w331s)

图 2 Advantech WebAccess 历年漏洞类型比例图

从图中可以看出 WebAccess 的历史漏洞类型比较丰富，包括了缓冲区溢出、SQL 注入、XSS、权限管理不当、敏感信息泄露、代码\\命令执行等。其中缓冲区溢出类型漏洞最多，占到漏洞总数的 1/3 以上；其次是权限管理类漏洞（11.94%）、敏感信息泄露 (8.96%)、SQL 注入（8.21%）。由此可以看出，WebAccess 的漏洞面较为广泛。

为了分析 WebAccess 漏洞类型的演变趋势，我们对其历年不同漏洞类型的数量进行了梳理，如图 3：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/70d2b5f1-51e7-448a-82a2-635905b874ba.jpg-w331s)

图 3 Advantech WebAccess 历年漏洞类型数量

从图中可以看出，WebAccess 的漏洞类型趋势没有明显变化，其中缓冲区溢出漏洞和权限管理漏洞在经历 2014 年井喷后仍然没有得到有效缓解，代码\\命令执行漏洞从 2018 年开始增多。

### Advantech WebAccess Node 多个 ZDI 漏洞分析

在 ZDI 披露的 Advantech WebAccess Node 的漏洞中，大部分都存在于 webvrpcs.exe 中的 RPC 通讯模块。RPC（Remote Procedure Call）即远程过程调用，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC 协议假定某些传输协议的存在，如 TCP 或 UDP，为通信程序之间携带信息数据。RPC 协议屏蔽了编程语言之间的特性差异，广泛应用于分布式系统中，在工业控制 DCS 系统中也被广泛应用。

对 Advantech WebAccess Node 分析发现，其采用 RPC 协议来实现 ODBC 和一些控制台命令。但在具体功能实现时缺乏相应的安全检查，导致产生了命令执行、内存破坏等多个漏洞。我们对这些漏洞进行了分析，其中典型漏洞（CVE-2017-16720、CVE-2019-10993、CVE-2019-10991）的分析如下。

#### 1、CVE-2017-16720 命令执行漏洞

在该漏洞位于 webvrpcs 程序的 RPC IOCTL code 0x2711 功能实现。在一定条件下可利用该漏洞在安装有 Advantech WebAccess 的系统上执行任意命令。该漏洞的 EXP 已被公开，核心代码如图 4 所示：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/76d0b2a4-3532-4c40-906a-b11e62578a4e.jpg-w331s)

图 4 CVE-2017-16720 利用代码

为了分析该漏洞的代码路径，先用 IDA 打开 webvrpcs.exe，然后通过 mida 插件提取出其 RPC 接口，可以看到 opcode 0x1 对应的处理函数为 sub_401260。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/8839f1ce-ccb0-424c-a04b-bfb6adc7d324.jpg-w331s)

图 5 webvrpcs.exe 使用 IDA mida 插件分析 RPC 接口

sub_401260（图 6 所示伪代码）首先对 RPC 消息的头部数据进行处理，然后调用 sub_402c60 函数。sub_402c60 函数最终调用 DsDaqWebService 函数（如图 7 所示），而 DsDaqWebService 函数源于动态链接库 drawsrv.dll。用 IDA 打开 drawsrv.dll 并定位到函数 DsDaqWebService，DsDaqWebService 函数实现了各个不同 IOCTL code 的功能，而 0x2711 对应的处理函数为 sub_100017B0。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/222b9cd2-579e-4d44-95eb-c16a0f23cf89.jpg-w331s)

图 6 函数 sub_401260 伪代码

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/e136ab0e-993a-4269-9f7c-e0d6027becdb.jpg-w331s)

图 7 函数 DsDaqWebService 伪代码

分析 sub_100017B0（图 8 所示），该函数中调用了 CreateProcessA() 函数创建进程。其中 lpCommandLine 参数由 RPC 客户端发送，且此处未对此参数进行任何检查。因此，可以通过控制该参数使得 CreateProcessA() 执行任意命令，从而导致远程命令执行。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/b250f8f1-9571-46ca-b60c-f27e348af80e.jpg-w331s)

图 8 漏洞函数 sub_100017B0 执行伪代码

该漏洞在 WebAccess Node 8.4 中得到修复，如图 9 所示 (case 0x2711 为补丁前的分支代码，case 1 为补丁后的分支代码) ，修复后的 IOCTL 增加了 sub_100022D0 函数对用户的输入内容进行检查。具体的检查方式是同白名单进行比较来判断参数是否合法。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/0f083b52-71ca-4011-b020-deb93f4602d2-163054812735410.jpg-w331s)

图 9 IOCTL code 0x2711 补丁前后伪代码

同样类型的漏洞还有 CVE-2019-10985，该漏洞触发点位于 IOCTL code 0x2715 功能实现。该功能是通过 unlink 删除指定文件，而文件名可以由 RPC 客户端任意指定。由于对参数没有任何过滤，可以设置该参数为任意文件路径，从而导致任意文件删除漏洞。

#### 2、CVE-2019-10993 指针解引用漏洞

该漏洞同样也是位于 webvrpcs 处理 RPC 消息的动态链接库 drawsrv.dll 中，触发点在处理 IOCTL code 0x27DB 的代码中。如下图 10 所示：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/2d102436-1cd4-4b3d-99cf-4ac3d743ed27.jpg-w331s)

图 10 IOCTL code 0x27DB 伪代码

通过分析，Filename 变量直接来源于 webvrpcs 接收的 RPC 数据，通过修改该 Filename 的值可以控制 SQLFreeConnet 的参数。分析 SQLFreeConnet 函数，其调用了函数 FreeIDbc 释放连接句柄 ConnectionHandle（即 SQLFreeConnet 参数）。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/0f66c7e2-3396-4e55-a3ab-b8da97426066.jpg-w331s)

图 11 IOCTL code 0x27DB 处理流程

在 FreeIDbc 函数中，ConnectionHandle 会被视为指针类型解引用。因此，通过构造 RPC 消息可控制指针解引用，从而造成内存访问错误。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/d452e2a2-7358-408f-9891-fe92637d3402.jpg-w331s)

图 12 FreeIDbc 函数处理流程

进一步分析可发现，参数 Memory 赋值给了 v2，在后续的操作中 (v2\[1]+36) 赋值给了 v7，后又进行了 v7\[1]的操作。而 Memory 受用户输入控制，因此 v7\[1]写入内存的地址可控，从而形成一个任意地址写漏洞。由于是内存写入操作，精心利用可能造成代码执行。

类似这类指针未校验的漏洞在 Advantech WebAccess Node 中还有很多，CVE 编号均为 CVE-2019-10993，包含多个 ZDI 漏洞编号 (ZDI-19-614 ZDI-19-613 ZDI-19-612 ZDI-19-611 ZDI-19-610 ZDI-19-609 ZDI-19-608 ZDI-19-607 ZDI-19-606 ZDI-19-605 ZDI-19-604 ZDI-19-603 ZDI-19-602 ZDI-19-601 ZDI-19-600 ZDI-19-599 ZDI-19-598 ZDI-19-597 ZDI-19-596 ZDI-19-595)。实际上这类漏洞就是对传入 webvrpcs 中的 DsDaqWebService 函数的参数没有进行检查，导致几乎所有的 IOCTL code ODBC 类函数都可以产生内存破坏漏洞，如图 13 所示：

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/19e63719-036d-459f-85e5-cba59bd959c1.jpg-w331s)

图 13 可能造成内存破坏的 IOCTL code

该系列漏洞在新版本 8.4 中得到修复，方式为在调用 SQL 函数前增加了检验函数 (链式校验流程如图 15)，判断参数是否合法，如图 14 所示。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/fd65c651-e0e2-4dcc-b7b3-6691accb086a.jpg-w331s)

图 14 修复后的 DsDaqWebService 函数

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/ef3e6e19-5094-4dcb-8b30-bb24b56163ed.jpg-w331s)

图 15 链式校验流程

**3、CVE-2019-10991 栈溢出漏洞**

CVE-2019-10991 包含了多个栈溢出漏洞，由于这些漏洞的触发条件和产生原因都较为相似，在此仅对其中一个漏洞（ZDI-19-594）进行详细分析。

WebAccess Node 软件包含了一系列小组件程序，bwscrp.exe 是其中一个。bwscrp.exe 程序入口函数接收两个命令行参数（如图 16），随后使用命令行参数 lpCmdLine 作为函数参数之一调用了函数 sub_4061F0。在函数 sub_4061F0 中，Source 即为 lpCmdLine，后续执行直接使用 strcpy 将 Source 拷贝到栈参数 Dest，而没有检查 Source 的数据长度。由于变量 Dest 距离栈顶只有 400 字节，当 Source 长度超过 404 字节时， strcpy 函数调用将覆盖栈上的函数返回地址，是一个典型的栈溢出漏洞。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/91d7225b-43dc-4348-853b-e3d242cd5653.jpg-w331s)

图 16 bwscrp.exe WinMain 函数伪代码

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/3c13dbe7-76e8-4ac6-b655-a417b85ae03c.jpg-w331s)

图 17 bwscrp sub_4061F0 函数伪代码

此外，CVE-2019-6550 (ZDI-19-585 ZDI-19-330 ZDI-19-329 ZDI-19-328 ZDI-19-327 ZDI-19-325 ZDI-19-323) 漏洞原理同 CVE-2019-10991 也极为类似。CVE-2019-6550(ZDI-19-330) 栈溢出漏洞存在于 upandpr.exe。该程序主函数中的 scanf 调用将用户提供的数据拷贝到栈内存，在拷贝之前未对用户提供的数据长度进行验证，如图 18 所示。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/6dc7be3c-be62-434d-b0ab-c8b60947b041.jpg-w331s)

图 18 upandpr.exe WinMain 函数伪代码

CVE-2019-10991 在最新版本的 Advantech WebAccess Node 中得到修复，补丁方式大致相同。以图 19 所示的 ZDI-19-594 为例 (左部分为补丁前代码，右部分为补丁后代码)，修复方式为在 strcpy 调用之前加入数据长度检查。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/ae5fb095-821d-4289-bbaa-32d33402a0be.jpg-w331s)

图 19 bwscrp.exe 补丁前后反编译伪代码

CVE-2019-6550 在最新版本的 Advantech WebAccess Node 中也得到修复，补丁方式大致相同。以图 20 所示的 ZDI-19-330 为例 (上部分为补丁前代码，下部分为补丁后代码)，修复方式为在 sscanf 的格式化输出符设置了最大字符长度来防止 sscanf 函数栈溢出的产生。

![](Advantech%20WebAccess%20%E5%A4%9A%E4%B8%AA%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90.assets/b15d46b4-6746-42d3-860a-7cbf5581e759.jpg-w331s)

图 20 upandpr.exe 补丁前后反编译伪代码

### Advantech WebAccess Node 漏洞挖掘

在分析 ZDI 披露的漏洞过程中，我们对 Advantech WebAccess 进行了初步审计，额外发现了三个漏洞，可导致任意文件删除和远程命令执行。

#### 1、CNVD-2019-23511 任意文件删除漏洞（中危）

WebAccess Node 软件会在系统中注册一个动态模块，分析该模块发现其包含一个文件删除函数，但没有对传入参数的进行安全检查过滤，导致存在任意文件删除漏洞。

#### 2、CNVD-2019-23512 命令执行漏洞（高危）

WebAccess Node 软件会在系统中注册另一个动态模块，分析该模块发现其包含一个外部程序调用功能，但没有对传入的调用参数进行检查，导致存在任意命令执行漏洞。

#### 3、CNVD-2019-23513 命令执行漏洞（高危）

在分析 ZDI 披露的漏洞过程中，我们对 Advantech WebAccess 进行了初步审计，额外发现了三个漏洞，可导致任意文件删除和远程命令执行。

### 结 语

通过对这一系列漏洞的分析可以发现，Advantech WebAccess 软件在实现过程中缺乏对程序输入的安全检查代码，对重要操作的认证不足，因此才爆出如此多的漏洞。不同于常规信息化系统，工业控制系统对稳定性的要求极高，工控软件漏洞被利用可能造成严重的后果。希望 WebAccess 相关用户单位持续关注其漏洞公告，及时安装补丁以修复相关漏洞。

**参考链接：** 

1\.[https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=webaccess](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=webaccess "https&#x3A;//cve.mitre.org/cgi-bin/cvekey.cgi?keyword=webaccess")

2\.[https://www.zerodayinitiative.com/advisories/ZDI-18-024/](https://www.zerodayinitiative.com/advisories/ZDI-18-024/ "https&#x3A;//www.zerodayinitiative.com/advisories/ZDI-18-024/")

3\.[https://www.exploit-db.com/exploits/44278](https://www.exploit-db.com/exploits/44278 "https&#x3A;//www.exploit-db.com/exploits/44278")

4\.[https://www.zerodayinitiative.com/advisories/ZDI-19-616/](https://www.zerodayinitiative.com/advisories/ZDI-19-616/ "https&#x3A;//www.zerodayinitiative.com/advisories/ZDI-19-616/")

5\.[https://www.zerodayinitiative.com/advisories/ZDI-19-594/](https://www.zerodayinitiative.com/advisories/ZDI-19-594/ "https&#x3A;//www.zerodayinitiative.com/advisories/ZDI-19-594/")

* * *

 本文由 Seebug Paper 发布，如需转载请注明来源。本文地址：[https://paper.seebug.org/1017/](https://paper.seebug.org/1017/)

[← 分析 Curl-P 以及攻击 IOTA 加密货币](/1015/) [漏洞扫描技巧篇 \[Web 漏洞扫描器\] →](/1018/)

['s Picture](/users/author/?nickname=%E5%90%AF%E6%98%8E%E6%98%9F%E8%BE%B0ADLab)r

#### [启明星辰 ADLab](/users/author/?nickname=%E5%90%AF%E6%98%8E%E6%98%9F%E8%BE%B0ADLab)

ADLab 成立于 1999 年，是中国安全行业最早成立的攻防技术研究实验室之一，微软 MAPP 计划核心成员，“黑雀攻击” 概念首推者。截止目前，ADLab 已通过 CVE 累计发布安全漏洞 1000 余个，通过 CNVD/CNNVD 累计发布安全漏洞 900 余个，持续保持国际网络安全领域一流水准。实验室研究方向涵盖操作系统与应用系统安全研究、移动智能终端安全研究、物联网智能设备安全研究、Web 安全研究、工控系统安全研究、云安全研究。研究成果应用于产品核心技术研究、国家重点科技项目攻关、专业安全服务等。

阅读更多有关[该作者](/users/author/?nickname=%E5%90%AF%E6%98%8E%E6%98%9F%E8%BE%B0ADLab)的文章

 [https://paper.seebug.org/1017/](https://paper.seebug.org/1017/)
