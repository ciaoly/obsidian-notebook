# 2020年全国网络与信息安全管理职业技能大赛B卷操作题writeup - _nul1 - 博客园
# [2020 年全国网络与信息安全管理职业技能大赛 B 卷操作题 writeup](https://www.cnblogs.com/nul1/p/13843913.html)

**2020 年全国行业职业技能竞赛  
全国网络与信息安全管理职业技能大赛操作技能赛题（B 卷）**

内容

* * *

1.  项目介绍
2.  选手指南
3.  任务目标

* * *

### 简介

> 你和你的团队是 A 公司的网络与信息安全运维及保障技术支撑队伍，负责 A 公司的企业内 OA 以及互联网论坛的运维工作，根据信息系统重要程度及行业监管要求，定为等级保护 3 级系统。现在 A 公司要求分别根据不同的任务完成网络与信息安全防护、管理及对于网络安全事件应急处置工作。

### 任务描述

> 本项目需要运用不同的网络与信息安全技术，任务有以下部分：  
> 网络与信息安全防护  
> 网络与信息安全管理  
> 网络与信息安全处置  
> 选手必须按照比赛要求对网络基础设施进行防护和管理，对一些网络与信息安全事件进行应急处理。所有的设备和服务可以正常运作，但尚未采取任何安全防护措施。在项目中，会给出一部分相对直接的安全防护和管理实施要求，一部分则为开放式操作要求。选手需要在设备条件的限制下尽力满足所有的比赛要求并符合行业规范操作。

### 选手指南

> 1.  在开始配置前详细阅读所有的任务。每一项任务都可能和前后任务的完成有所依赖。依赖于上一项或下一项的完成。
> 2.  本次比赛为在线远程访问形式，在比赛前请确认设备是否能够正常访问，在比赛中请时刻注意配置操作是否会影响系统的正常访问，如在比赛时因配置原因造成系统无法正常访问，需要选手自行解决。
> 3.  比赛以在答题平台上提交 Flag 形式进行，Flag 必须与答题平台内对应序号答题框的预设完全匹配才能得分，所有 Flag 字符以小写、去除空格形式提交。如解题答案为： Flag{} 样例为：/etc/apache2/apache2.conf ServerSignature Off，则 Flag 提交：/etc/apache2/apache2.confserversignatureoff
> 4.  每题率先答出的前三位选手将有 15%、10% 和 5% 的额外奖励分，最后按比例折算百分制计入成绩。
> 5.  比赛可采用任何自带设备和工具，但禁止使用影响他人比赛或妨碍平台运行的工具、方法和手段，如有发现将立即取消成绩。  
>     所需设备，安装和材料；选手自行准备工具完成本项目所有要求实施的内容。

### 任务目标

* * *

项目基本信息：

1.  所有设备和虚拟机的主机名称根据提供的图示已预先配置；
2.  虚拟机已根据以下角色预先安装：  
    虚拟机  
    所提供的服务  
    Webserver Apache web server(OA 系统 8080 端口、论坛 80 端口) ，Snort，Mysql，Firewalld，ntp

### 基本信息

> **OA 系统**  
> 监听端口：8080；管理地址：[http://IP:8080/?m=login；管理用户名：admin；管理密码：qwer1234；所在本地目录：/var/www/oa](http://IP:8080/?m=login%EF%BC%9B%E7%AE%A1%E7%90%86%E7%94%A8%E6%88%B7%E5%90%8D%EF%BC%9Aadmin%EF%BC%9B%E7%AE%A1%E7%90%86%E5%AF%86%E7%A0%81%EF%BC%9Aqwer1234%EF%BC%9B%E6%89%80%E5%9C%A8%E6%9C%AC%E5%9C%B0%E7%9B%AE%E5%BD%95%EF%BC%9A/var/www/oa)

> **BBS 论坛**  
> 监听端口：80；管理地址：[http://IP/admin.php；管理用户名：admin；管理密码：1qaz2wsx#EDC；所在本地目录：/var/www/html](http://IP/admin.php%EF%BC%9B%E7%AE%A1%E7%90%86%E7%94%A8%E6%88%B7%E5%90%8D%EF%BC%9Aadmin%EF%BC%9B%E7%AE%A1%E7%90%86%E5%AF%86%E7%A0%81%EF%BC%9A1qaz2wsx#EDC%EF%BC%9B%E6%89%80%E5%9C%A8%E6%9C%AC%E5%9C%B0%E7%9B%AE%E5%BD%95%EF%BC%9A/var/www/html)  
> Mysql  
> 管理用户名：root 管理密码：root  
> Snort/snort_src

## 第一部分：网络与信息安全防护

**总体要求：根据 A 企业业务需求，对 A 企业的网络、系统及应用配置主动及被动防御系统。** 

1.  公司系统未采用严格的过滤策略对外部访问流量进行过滤，根据公司业务架构和需求， 修改 firewalld 的配置文件，为 http 服务增加一个 8080 端口，提交修改的命令。（去除空格）；Flag{} 样例：  
    &lt;portprotocol="tcp"port="8080"/>
2.  公司系统未采用严格的过滤策略对外部访问流量进行过滤，根据公司业务架构和需求， 配置系统 firewalld 的策略，永久开放必要的网络服务，提交配置命令，每一条正确命令为一个 flag（命令中应包含 zone、service 并去除空格，依据服务字母排序提交）； Flag{} 样例：command --key=value --key=value --key  
    firewall-cmd--zone=public--add-service=http--permanent
3.  公司系统未采用严格的过滤策略对外部访问流量进行过滤，根据公司业务架构和需求， 配置系统 firewalld 的策略，永久开放必要的网络服务，提交配置命令，每一条正确命令为一个 flag（命令中应包含 zone、service 并去除空格，依据服务字母排序提交）； Flag{} 样例：command --key=value --key=value --key  
    firewall-cmd--zone=public--add-service=https--permanent
4.  安装配置 Snort，在 community3.rules 规则文件中找到能够响应给定测试用例的对应规则，提交规则中的 msg 字段；测试用例：[http://IP/forum.php?error=](http://IP/forum.php?error=)  
    malware-cncwin.backdoor.jraterrorinputdetected
5.  检查系统运行的应用服务，使用 systemctl 命令关闭 docker 开机自启，命令作为 flag 提交；  
    systemctldisabledocker
6.  Lynis 2.1.1 对系统进行全盘扫描，提交扫描结果警告为 BOOT-5184 的文件绝对路径；  
    /etc/rc.local
7.  启用 apache ssl 模块，对所有访问论坛的请求设置 https 强制跳转，通过 http 访问该业务进行测试，查看 apache 日志，提交日志中的 user agent 字段（包含 OpenSSL 关键字）；
8.  配置 Modsecurity ，启用 REQUEST-930-APPLICATION-ATTACK-LFI.conf 规则，利用所给的攻击测试用例测试，提交 Modsecurity 生成日志中第一条 message 后的 warning 字段 (需要提交最开始的 Warning.)；测试用例：[http://IP/forum.php?id=/etc/passwd](http://IP/forum.php?id=/etc/passwd)

## 第二部分：网络与信息安全管理

**总体要求：根据 A 公司业务需求，设置系统权限、数据库权限的分级管理。** 

1.  配置口令更换周期为 90 天，以配置文件绝对路径和配置命令为 flag 提交；Flag{} 样例：/path/filename key value  
    /etc/login.defspass_max_days90
2.  配置口令更改最小间隔天数为 6 天，以配置文件绝对路径和配置命令为 flag 提交；Flag{} 样例：/path/filename key value  
    /etc/login.defspass_min_days6
3.  配置口令过期前警告天数为 30 天，以配置文件绝对路径和配置命令为 flag 提交；Flag{} 样例：/path/filename key value  
    /etc/login.defspass_warn_age30
4.  导入环境变量，配置当前会话超时 10 分钟自动退出，以配置语句作为 flag 提交；Flag{} 样例：command key=value  
    exporttmout=600
5.  检查 / var/log / 目录下的文件权限，找到存在具有明显不合理权限的文件，还原该文件权限为默认状态，将该文件名和 ls -l 文件名 | awk '{print $1}' 作为 flag 提交；Flag{} 样例：filename key  
    auth.log-rw-r-----
6.  安装配置 tripwire，对 / var/log/apache2 目录进行监控, 级别为 SEC_LOG，依次执行测试语句 chown www-data:www-data /var/log/apache2/access.log ；tripwire --check 后将 Object Summary 中涉及 / var/log/apache2/access.log 文件改变的字段依次拼接作为 flag 提交；Flag{} 样例：key:"/path/filename"
7.  检查并取消数据库非管理员用户查询所有用户命令执行信息的权限，以 show grants for 用户名 语句查询结果作为 flag 提交；
8.  mysql 的命令历史文件中会包含许多历史操作记录，如登录时将密码直接跟在 - p 参数后，则会将明文密码记录到历史文件中。为防止敏感信息泄漏，删除该文件，给以该文件的绝对路径作为 flag 提交；  
    /root/.mysql_history
9.  隐藏 APACHE 版本信息号，只返回服务器名。将修改后的对应行作为 flag 提交（参数值简写）；  
    Flag{} 样例：key value  
    servertokensprod
10. 通过 logrotate.conf 设置日志存留时间为 6 个月，以配置参数为 flag 提交；Flag{} 样例：key key value  
    monthlyrotate6

## 第三部分：网络与信息安全处置

**总体要求：在接到 A 公司员工反映在 OA 系统中收到不明邮件后， A 公司立即启动应急响应，经初步分析，发现公司部分应用已被挂马，业务系统（论坛、OA）疑似遭受黑客攻 击，并由可能已经发生了数据泄漏，请您协助对本次网络攻击事件进行追踪溯源，分析黑客的攻击手段、查找 WEB 应用及系统存在的漏洞。同时根据本次事件的调查分析结果，对存在的漏洞进行修复，确认并删除黑客隐藏在系统中的后门，恢复系统的正常运行。** 

1.  提交最有可能是攻击者第一次爆破论坛后台密码的时间；  
    Flag{} 样例：11/Sep/1999:01:01:01  
    21/aug/2020:09:14:29
2.  提交攻击者留下系统后门的主进程完整启动命令；  
    /usr/update
3.  提交攻击者留下系统后门的主进程实际调用文件的绝对路径；  
    /var/www/html/data/cache/update
4.  找到并提交攻击者留下的提权文件的绝对路径；
5.  提交攻击者对网站利用的 CVE；
6.  提交攻击者成功留下小马的时间，精确到秒；  
    Flag{} 样例：1990-01-01 20:20:20  
    2020-08-2111:34:00
7.  提交攻击者第一次使用后门管理工具连接小马的时间；  
    Flag{} 样例：11/Sep/1999:01:01:01  
    21/aug/2020:11:39:57
8.  提交攻击者留下的攻击大马的绝对路径；  
    /var/www/html/data/cache/cache_config.php
9.  提交最有可能是攻击者操作系统的信息；
10. 提交攻击者留下的黑页中的 flag； Flag{} 样例：flag{xxx}  
    flag{90b058411942bb8909ab52dec800ae5c}
11. 提交可能被攻击者脱库的数据库名称, 按首字母排序提交；
12. 发现被入侵后，开始捕获流量, 这时候的攻击者 IP；  
    114.86.226.190
13. 反弹 shell 回连成功后执行的第一条命令；  
    id
14. 攻击者在目标服务器上留了一个 ssh 后门，并且实现了自启动，ssh 后门自启动写入对应数据包的 Arrival Time；  
    Flag{} 样例：Jun 01, 2000 01:01:01.00000001 CST
15. 恢复攻击者上传的美女图片，提取里面违法信息，提交恶意网址；
16. 恢复攻击者上传的枪支毒品视频，提取里面的隐藏信息；
17. 攻击者发布的枪支贩卖信息中包含的有害链接；
    [https://www.cnblogs.com/nul1/p/13843913.html](https://www.cnblogs.com/nul1/p/13843913.html)
