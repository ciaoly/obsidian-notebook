# 浅谈Linux主机加固 - FreeBuf网络安全行业门户
浅谈 Linux 主机加固

[雷石安全实验室](https://www.freebuf.com/author/雷石安全实验室) 2020-09-22 01:13:23 214537 4

## ​前 言

Linux 系统被应用于大部分企业的服务器上，因此在等保测评中主机加固也是必须要完成的一项环节。

由于在之后项目开始要进行主机加固，因此对 linux 的加固流程进行总结学习。

Linux 的主机加固主要分为：账号安全、认证授权、协议安全、审计安全。简而言之，就是 4A(统一安全管理平台解决方案)。

这边就使用我自己 kali 的虚拟机进行试验学习。

## 一、账户安全

### 1、口令生存期

gedit /etc/login.defs

![](https://image.3001.net/images/20200922/1600708089_5f68ddf996484f612e5b1.png!small)

在此处对密码的长度、时间、过期警告进行修改

> PASS_MAX_DAYS   90  #密码最长过期天数
>
> PASS_MIN_DAYS   10   #密码最小过期天数
>
> PASS_WARN_AGE   7   #密码过期警告天数

如果修改设置有最小长度也需要修改

> PASS_MIN_LEN    8   #密码最小长度

### 2、口令复杂度（很重要)

password requisite  pam_cracklib.so

在文件中找到

password requisite  pam_cracklib.so

将其修改为:

password requisite  pam_cracklib.so try_first_pass retry=3 dcredit=-1 lcredit=-1 ucredit=-1 ocredit=-1 minlen=8

备注：至少包含一个数字、一个小写字母、一个大写字母、一个特殊字符、且密码长度 >=8

### 3、版本信息

cat /proc/version

![](https://image.3001.net/images/20200922/1600708103_5f68de07b8641d2b6151f.png!small)

### 4、限制 xx 用户登录

/etc/hosts.deny

![](https://image.3001.net/images/20200922/1600708116_5f68de148d8306434203b.png!small)

添加内容：

> sshd : 192.168.1.1

禁止 192.168.1.1 对服务器进行 ssh 的登陆

### 5、检查是否有其他 uid=0 的用户

awk -F “：” '($3==0)  {print  $1}' /etc/passwd

6、登陆超时限制

cp -p /etc/profile /etc/profile_bak(备份)

gedit /etc/profile

![](https://image.3001.net/images/20200922/1600708129_5f68de21abd0883b97e5a.png!small)

增加

TMOUT=300

export TMOUT

或者

echo 'export TMOUT=300'>>/etc/profile

echo 'readonly TMOUT' >>/etc/profile

source /etc/profile

### 7、检查是否使用 PAM 认证模块禁止 wheel 组之外的用户 su 为 root

gedit /etc/pam.d/su

新增

auth          sufficient     pam_rootok.so

auth          required     pam_wheel.so use_uid

备注：auth 与 sufficient 之间由两个 tab 建隔开，sufficient 与动态库路径之间使用一个 tab 建隔开

### 8、禁用无用账户

cat /etc/passwd #查看口令文件，确认不必要的账号。

![](https://image.3001.net/images/20200922/1600708173_5f68de4db64d0470ba885.png!small)

passwd -l user # 锁定不必要的账号

### 9、账户锁定

gedit /etc/pam.d/system-auth

在文件中修改或者添加

auth required pam_tally.so onerr=fail deny=3 unlock_time=7200

锁定账户举例：

passwd -l bin

passwd -l sys

passwd -l adm

10、检查系统弱口令

john /etc/shadow --single

john /etc/shadow --wordlist=pass.dic

我这边有报错 就不展示了

使用 passwd 用户 命令为用户设置复杂的密码

## 二、协议安全

### 1、openssh 升级（按需做）

yum update  openssh

### 2、定时任务（防止病毒感染）

定时任务检查：

crontab -l

一次性任务检查：

at -l

### 3、限制 ssh 登录（看是否需要）

首先 cat /etc/ssh/sshd_config

查看 PermitRootLogin 是否为 no

![](https://image.3001.net/images/20200922/1600708194_5f68de62d6497a7e615e6.png!small)

gedit  /etc/ssh/sshd_config

PermitRootLogin no 不允许 root 登陆

Protocol 2 修改 ssh 使用的协议版本

MaxAuthTries 3 修改允许密码错误次数

或 echo "tty1" > /etc/securetty

hmod 700 /root

### 4、限制 su 为 root 用户

gedit /etc/pam.d/su

![](https://image.3001.net/images/20200922/1600708222_5f68de7ea6790a364d64c.png!small)

在头部添加 auth required /lib/security/pam_wheel.so group=wheel

### 5、禁止 root 用户登录 ftp

因为我的 kali 下没有这个文件，因此借鉴一下网上的

cat /etc/pam.d/vsftpd

Auth required pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed

\#其中 file=/etc/vsftpd/ftpusers 即为当前系统上的 ftpusers 文件.

echo  “root”   >>   /etc/vsftpd/ftpusersv

### 6、防止 flood 攻击

gedit  /etc/sysctl.conf

![](https://image.3001.net/images/20200922/1600708236_5f68de8cbd6e2020bdb53.png!small)

增加 net.ipv4.tcp_syncookies = 1

然后 sysctl  -p

### 7、禁 ping

echo 0 > /proc/sys/net/ipv4/icmp_echo_igore_all

### 8、检查异常进程

ps aux|sort -rn -k +3|head

\#检查 cpu 占用前 10

ps aux|sort -rn -k +4|head

\#检查内存占用前 10

### 9、关闭无效的服务及端口

比如邮箱

> service postfix status
>
> chkconfig --del postfix
>
> chkconfig postfix off

比如 cpus

> service cups status
>
> chkconfig --del cups
>
> chkconfig cups off

### 10、设置防火墙策略

或者用防火墙策略：

service iptables status

echo '请根据用户实际业务端口占用等情况进行设置!'

例如：

gedit /etc/sysconfig/iptables 添加如下策略

\-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

\-A INPUT -m state --state NEW -m udp -p udp --dport 8080 -j ACCEPT

以下举例：

iptables -I INPUT -s 22.48.11.11 -j DROP

\# 22.48.11.11 的包全部屏蔽

iptables -I INPUT -s 22.48.11.0/24 -j DROP

\#22.48.11.1 到 22.48.11.255 的访问全部屏蔽

iptables -I INPUT -s 192.168.1.1 -p tcp --dport 80 -j DROP

\# 192.168.1.1 的 80 端口的访问全部屏蔽

iptables -I INPUT -s 192.168.1.0/24 -p tcp --dport 80 -j DROP

\#192.168.1.1 到 192.168.1.255 的 80 端口的访问全部屏蔽

service iptabels restart

\#重启防火墙

### 11、设置历史记录数量

cp /etc/profile /etc/profile_xu_bak(备份)

sed -i s/'HISTSIZE=1000'/'HISTSIZE=5000'/g /etc/profile(修改)

cat /etc/profile |grep HISTSIZE|grep -v export(检查)

## 三、认证权限

### 1、配置用户最小权限

> chmod 644 /etc/passwd
>
> chmod 400 /etc/shadow
>
> chmod 644 /etc/group

### 2、文件与目录缺省权限控制

> cp /etc/profile /etc/profile.bak(备份)
>
> gedit  /etc/profile

增加

> umask 027
>
> source  /etc/profile

## 四、日志审计

### 1、启用远程日志功能

gedit /etc/rsyslog.conf

![](https://image.3001.net/images/20200922/1600708259_5f68dea320430c824d957.png!small)

\*.\*         @Syslog 日志服务器 IP

\### 注意：\*和 @之间存在的是 tab 键，非空格。

### 2、检查是否记录安全事件日志

gedit  /etc/syslog.conf 或者 /etc/rsyslog.conf

在文件中加入如下内容:

\*.err;kern.debug;daemon.notice     /var/log/messages

chmod 640 /var/log/messages

service rsyslog restart

### 3、日志保留半年以上

cp/etc/logrotate.conf /etc/logrotate.conf_xu_bak(备份)

sed -i s/'rotate 4'/'rotate 12'/g /etc/logrotate.conf(修改)

service syslog restart(重启)

cat /etc/logrotate.conf |grep -v '#' |grep rotate(检查)

本文作者：雷石安全实验室， 转载请注明来自[FreeBuf.COM](https://www.freebuf.com)

\# linux 安全 # 主机 # 主机安全 
 [https://www.freebuf.com/articles/system/250501.html](https://www.freebuf.com/articles/system/250501.html)
