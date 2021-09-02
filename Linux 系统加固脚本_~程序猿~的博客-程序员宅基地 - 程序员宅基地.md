# Linux 系统加固脚本_~程序猿~的博客-程序员宅基地 - 程序员宅基地
## Linux 系统加固脚本\_~ 程序猿~ 的博客 - 程序员宅基地

Linux 系统加固脚本

1\. 用户的密码必须要包含：大小写字母、数字、特殊符号  
2. 用户的密码长度不少于 10 位  
3. 用户密码的过期时间：30 天  
4. 密码过期前 8 天提醒用户  
5. 用户密码输入错误 3 次后，锁定用户 10 分钟  
6. 用户 1 分钟内没有操作，自动注销！

\#!/bin/bash  
#增加备用的 root 用户

\#新建用户的密码复杂度、长度  
pw_file=/etc/security/pwquality.conf  
login_file=/etc/login.defs  
auth_file=/etc/pam.d/system-auth

\#备份配置文件  
cp $pw_file $pw_file.bak  
cp $login_file $login_file.bak  
cp $auth_file $auth_file.bak

\#修改密码长度为 10 位  
echo minlen = 11 > /etc/security/pwquality.conf  
#新建密码至少包含 1 个数字  
echo dcredit = -1 >> /etc/security/pwquality.conf

\#新建密码至少包含 1 个小写字母  
echo lcredit = -1 >> /etc/security/pwquality.conf

\#新建密码至少包含 1 个大写字母  
echo ucredit = -1 >> /etc/security/pwquality.conf

\#新建密码至少包含 1 个特殊字符  
echo ocredit = -1 >> /etc/security/pwquality.conf

\#密码过期时间 30 天  
sed -i ‘/PASS_MAX_DAYS/s/99999/30/’ $login_file

\#过期前 7 天通知用户  
sed -i ‘/PASS_WARN_AGE/s/7/8/’ $login_file

\#用户输入密码错误 3 次, 封锁用户 10 分钟  
grep -q ‘pam_tally2.so’ /etc/pam.d/system-auth  
if \[ $? -ne 0 ];then  
echo ‘auth required pam_tally2.so deny=3 unlock_time=300 even_deny_root root_unlock_time=300’ >> /etc/pam.d/system-auth  
fi

\#用户 10 秒没有操作，则退出终端  
grep -q ‘TMOUT’ /etc/profile || echo “export ‘TMOUT=10’” >> /etc/profile

[](https://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/weixin_51596958/article/details/111151860](https://blog.csdn.net/weixin_51596958/article/details/111151860)

[原作者删帖](/copyright.html#del)   [不实内容删帖](/copyright.html#others)   [广告或垃圾文章投诉](# "投诉本文含广告或垃圾信息")

* * *

### 智能推荐

### [Linux 系统一键安全加固 shell 脚本及使用说明. rar](/article/qq453198/14950399)

Linux 系统一键安全加固 shell 脚本及使用说明。已经在 centos7 上验证后的，欢迎大家下载。

### [Linux 加固脚本](/article/lt_CSVN/95475758)

最近使用 Linux 搭建服务器，莫名奇妙的中了挖矿的病毒，导致 CPU 使用量到达 100%，项目都无法启动了，并且清除程序和病毒脚本后，过几天又出现了，所以将此加固的步骤记录一下： 服务器重新部署后，先新增一个用户，...

### [Linux 自动加固脚本](/article/weixin_39647062/10165170)

详细介绍 Linux 脚本加固方法，加固密码策略，账户策略等。

### [【Linux】Linux 安全加固脚本](/article/cubi56517/100274042)

\#安全加固脚本 #!/bin/bash #version1.0;writeat2017-03-29; #writebyRingoo; #1. 将 root 管理员名称修改为 Ringoo useradd...

### [15 个优秀开源的 Spring Boot 学习项目，一网打尽！](/article/u012702547/103506486)

Spring Boot 算是目前 Java 领域最火的技术栈了，松哥年初出版的 《Spring Boot + Vue 全栈开发实战》迄今为止已经加印了 8 次，Spring Boot 的受欢迎程度可见一斑。经常有人问松哥有没有推荐的 Spring Boot 学习...

### [2018 年 Linux 系统安全检查、加固 shell 脚本](/article/qq_35570677/10795102)

2018 年最新 Linux 系统安全检查、系统加固 shell 脚本，可过三级等保脚本。

### [linux 服务器安全加固 shell 脚本代码](/article/weixin_38731123/12847228)

有时候安装完服务器以后，需要一些安全设置，这段脚本就是为了安全加固所写，需要的朋友可以参考下

### [shell\_脚本\_linux\_安全加固](/article/qq_33020901/51591291)

关于 Linux 系统安全加固的具体实现脚本及基线检查规范，以供主机维护人员参考学习。 其中以下脚本主要实现的功能包括： \*加固项包括：密码长度、session 超时时间、删除不用的帐号和组、限制 root 用户直接 telnet 或...

### [Linux 系统安全加固](/article/weixin_44004462/108099777)

centos7 系统安全加固方案… 2 一. 密码长度与有效期… 2 默认配置：… 2 加固方案: 2 备注: 3 二. 密码复杂度… 3 默认配置: 3 加固方案: 3 备注: 4 三. 新口令不能与 4 个最近使用的相同… 4 默认配置: 4 加固方案: 5 ...

### [Python - 用于检查 Linux 内核配置中加固选项的脚本](/article/weixin_39840387/11518246)

用于检查 Linux 内核配置中加固选项的脚本

### [(20190419.v2)Linux 系统中的安全加固 --- 登陆安全](/article/GX_1_11_real/89334708)

总结一下，安全加固就是防止老婆给” 你 “戴帽子。 下面是从系统登陆的方面来总结的一些系统安全设置, 可根据需求设置。 【1】禁止 root 远程登录 禁止 root 用户，极易导致执行某些操作无权限；如使用此设置，请先...

### [Linux 基线检查，安全加固脚本](/article/qq_42568611/103757622)

\#!/bin/bash Author: 韩伟 Date: 2019-12-29 实现对用户密码策略的设定，如密码最长有效期等 date=date +%Y-%m-%d read -p “是否设置密码策略\[y/n]:” Y if \[ "Y"=="y"];thenread−p"设置密码最多可多少天不修改："A...

### [Linux 系统检测和防护脚本](/article/u014779378/103046483)

1、方便将服务器安全情况通过检测脚本直接输出 txt 文件，同时便于检查出安全隐患。 2、缩短安全检查和防护时间... 一、Linux 检测脚本 1.1、文件 压缩包包含 2 个文件: CentOS_Check_Script.sh : 脚本文件 README.txt :...

### [Linux 服务器安全加固 shell 脚本](/article/machen_smiling/8282733)

资深运维工程师亲自实践撰写的 Linux 服务器安全加固 shell 脚本，当然安全还要亲历亲为，不可马虎大意！

### [云服务器运维 - Linux 操作系统安全加固 / 防范黑客攻击](/article/qq_15071263/100186240)

文章目录云服务器运维 - Linux 操作系统安全加固 / 防范黑客攻击 1、帐号 1.1 禁用或删除无用账号 1.2 检查特殊账号 1.3 添加口令策略 1.4 限制用户 su1.5 禁止 root 用户直接登录 2、服务 2.1 关闭不必要的服务 2.2 SSH 服务安全 3...

### [Linux 系统安全防护加固](/article/qq_40907977/102929721)

服务器常见的攻击方法 1、针对端口进行渗透 2、odly 漏洞 ...4、针对 DDOS 攻击 服务器预防攻击 一、服务器 1，禁用 ROOT ...3，修改 ssh 的默认 22 端口 4，安装 DenyHosts 防暴力破解软件 ...1，禁用公网 IP 监听，包括 0.0.0.0 ...

### [Linux 系统安全加固 - openSSH 升级](/article/baidu_39459954/89447343)

本文在 2018 年 7 月 13 日亲测，安装环境 CentOS7 1、安装 telnet（） 为确保升级出现问题导致服务器无法连接，先安装 telnet 以备不时之需。 \[root@DZFP-DMZ-Server2 ~]# rpm -ivh telnet-0.17-64.el7.x86_64.rpm ...

### [运维工作梳理](/article/weixin_44980838/103607923)

1、什么是 linux 运维 运维是指大型组织已经建立好的网络软硬件的维护，就是要保证业务的上线与运作的正常。 在他运转的过程中，对他进行维护，他集合了网络、系统、数据库、开发、安全、监控于一身的技术。 ...

### [Linux 服务器安全加固](/article/qq_36119192/82906799)

Linux 用户方面的加固 设定密码策略​ 对用户密码强度的设定 对用户的登录次数进行限制 禁止 ROOT 用户远程登录 设置历史命令保存条数和账户超时时间 设置只有指定用户组才能使用 su 命令切换到 root 用户 对 Linux...

### [一个 Linux 系统安全设置的 Shell 脚本的分享（适用 CentOS）](/article/weixin_38519660/12846801)

主要介绍了一个设置 Linux 系统安全的 Shell 脚本的分享, 适用 CentOS, 包含大部份的安全设置, 只需执行脚本就可以得到一个相对安全的 Linux 系统了, 需要的朋友可以参考下

### [Linux 系统安全加固设置详细教程](/article/zl1zl2zl3/83614826)

1、如果是新安装系统， 一、对磁盘分区应考虑安全性： 1）根目录（/）、用户目录（/home）、临时目录（/tmp）和 / var 目录应分开到不同的磁盘分区； 2）以上各目录所在分区的磁盘空间大小应充分考虑，避免因某些...

### [【自动化脚本】Windows 操作系统安全加固](/article/shuy113/10016616)

本脚本为本人根据单位任务和个人兴趣编写，具有较好的交互操作，点击后不会直接执行操作，需要用户确认后才会执行，运行前会对注册表和组策略等信息进行备份，现上传分享给大家，如需对 Windows 操作系统安全进行加固...

### [18 项基线加固脚本](/article/pygain/9618502)

对 LINUX 下的 18 项基线安全加固

### [Linux 系统安全及应用加固———最适合新手学，新手都能看懂！超详细的理论 + 超详细的实验！呕心沥血之作完成...](/article/m0_46563938/107497220)

系统安全应用一、账号安全控制 1.1、账号安全基本措施 1.1.1、系统帐号清理 1.1.2、密码安全控制 1.1.3、命令历史限制 1.1.4、终端自动注销二、系统引导和登录控制 2.1、使用 su 命令切换用户 2.1.1、用途和用法 2.1.2、限制...

### [Linux 安全加固](/article/wz_cow/82049445)

1.1 锁定系统中多余的自建帐号 检查方法: 执行命令 #cat /etc/passwd #cat /etc/shadow 查看账户、口令文件，与系统管理员确认不必要的账号。对于一些保留的系统伪帐户如：bin, sys，adm，uucp，lp, nuucp，hp...

### [Linux 服务器加固方案](/article/qq_33168577/79585116)

对 Linux 服务器安全 和 防火墙的配置 和 服务器加固方案 进行简单的讲解，防止初级黑客的攻击，文章主要是针对小白，和运维新手，写的不好，可能有些出入，欢迎各位大佬指点。 一、适用 Linux 操作系统版本 1.Red ...

### [基线安全与 linux 基线加固方法](/article/pygain/52381453)

即安全基线配置，诸如操作系统、中间件和数据库的一个整体配置，这个版本中各项配置都符合安全方面的标准。比如在系统安装后需要按安全基线标准，将新机器中各项配置调整到一个安全、高效、合理的数值。 2. 基线扫描 ...

### [linux 安全加固](/article/sxliuxianlin/10811314)

linux 等保三级安全加固操作手册和说明文档，批量执行加固脚本。

### [基于 Java 语言的 SQL 注入和 XSS 攻击及加固](/article/CSDN讲师/6898)

本课程在 Java 语言的基础上，以 Web 应用安全为主题，分析 Sql 注入攻击的原理、XSS 攻击的原理；以及针对这两种攻击行为使用的加固方案。从而为政府或企业已有的 Web 应用提供安全解决方案参考。

### [等级保护 2.0 Linux 加固 Redhat 加固 基线](/article/m0_37203594/12994035)

Linux 的安全加固基线。从扫描器中导出的配置结果，提供了检查项目、配置命令等。能够为 Linux 提供较好的加固方案。

### 随便推点

 [https://www.cxyzjd.com/article/weixin_51596958/111151860](https://www.cxyzjd.com/article/weixin_51596958/111151860)
