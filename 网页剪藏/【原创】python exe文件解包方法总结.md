[LarryS](https://bbs.kanxue.com/homepage-600394.htm)

* * *

_注：_  
_除特别说明外，下面使用的python版本均为3.8.6_  
_文章中提到的相关工具均已包含在附件中。_

## 一、步骤

### 1. exe → pyc

#### 方法1：pyinstxtractor.py

执行`python pyinstxtractor.py <待解包文件名>` ，如果成功，即可获得`<待解包文件名>_extracted` 文件夹。

_注：执行时会提示python版本问题，想要正常解包必须使用正确的python版本_。

单文件版本: [pyinstxtractor/pyinstxtractor-ng: PyInstaller Extractor Next Generation (github.com)](https://github.com/pyinstxtractor/pyinstxtractor-ng)

#### 方法2：archive_viewer.py

执行`python archive_viewer.py <待解包文件名>` ，会打印EXE文件中包含的所有文件信息

使用`x <文件名>`命令将想要提取出的文件提取出来，`q` 命令退出。

#### 两者的区别

方法1可以一次性提取出所有文件，方法2只能逐个提取文件。但是在个人使用时，方法2的成功率相对较高。可以先尝试用方法1，失败后用方法2。

### 2. pyc → py

步骤1获得的文件是pyc文件，还需要进一步反编译获得py文件。

_注：我遇到过直接获得py文件的情况，所以在反编译之前可以先查看一下是不是已经成功了。_

#### 2.1 pyc文件恢复

_注：最新版pyinstxtractor.py支持自动恢复pyc，但是经实验不能保证准确性。或者说需要使用准确的python版本才行。_

在将python文件打包成exe文件的过程中，会抹去pyc文件前面的部分信息，所以在反编译之前需要检查并添加上这部分信息。

抹去的信息内容可以从`struct`文件中获取：

struct文件：  
![](https://bbs.kanxue.com/upload/attach/202012/600394_RVR8EMZYXH2BVCW.png)

pyc文件：  
![](https://bbs.kanxue.com/upload/attach/202012/600394_EZEJWJ4J4PC4JN8.png)

**Q1：需要添加多少字节？**

多个参考文章中提到的添加字节数都不一致，这应该与使用的python版本有关。但是在已知的几个例子中，可以看出`pyc`文件开头的几个字节与`struct`文件中的字节是一样的。比如说上图中`pyc`文件是以`E3 00 00 00`开头，而这部分字节就和`struct`文件的第二行起始字节相同。

因此在此例中，需要复制添加的字节就是`struct`文件中第一行的16个字节

**Q2：添加的方法是什么？**

在010editor中，选择`Edit→Insert/Overwrite→InsertBytes`，`Start Address`填0，`Size`填16。

然后将字节复制进去即可。

#### 2.2 反编译

**反编译工具**

1.  Easy Python Decompiler
    
    这是一个GUI界面的可执行文件，下载下来直接用就可以，但是并不是每次都能成功，有些magic value不能识别。
    
2.  uncompyle6
    
    该工具需要使用`pip`安装，使用脚本执行。成功率较高。
    

**反编译提示magic value有问题怎么办？**

在上面我们添加的16个字节中，前四个字节表示的就是magic value，其中前两个字节表示的是python的版本号，一般来说magic value的问题就是版本号有问题，编译工具没有识别出来该版本号表示的python版本。

如果使用的是Easy Python Decompiler，那么就可以直接转用`uncompyle6`了。

如果用的已经是`uncompyle6`，那么需要先看一下它可以识别的版本号都有哪些，这个信息可以在`xdis`包的`magics.py`文件中找到，具体方法如下：

1.  命令行输入`pip install xdis`，查看其安装位置
    
    ![](https://bbs.kanxue.com/upload/attach/202012/600394_BJEDVBYG9HPR6VK.png)
    
2.  到该目录下，打开xdis文件夹下的magics.py文件
    
3.  确定需要识别的版本号
    
    就是之前添加的16个字节的前两个字节，此例中为`0D42`，需要转换为十进制，就是`3394`。
    
4.  查看magics.py中是否有该版本号。
    
    我这里一开始没有发现这个版本号
    
    ![](https://bbs.kanxue.com/upload/attach/202012/600394_YKH89PB6D8FX66S.png)
    

**我的解决方法**

我在谷歌上搜索了`int2magic(3394)`，找到了[这个Github项目](https://github.com/rocky/python-xdis)。

下载下来，按照介绍进行安装。

在执行`pip install -e .`的时候，提示我`xdis`和`uncompyle6`的版本不匹配，于是卸载了`uncompyle6`，又重新安装了一次。

目前的版本：`xdis 5.0.5` `uncompyle6 3.7.4`

查看该版本`xdis`的`magics.py`文件：

![](https://bbs.kanxue.com/upload/attach/202012/600394_7Y95XYYTSBGEQCN.png)

可以看到已经有3394了。

反编译：

![](https://bbs.kanxue.com/upload/attach/202012/600394_MTMA25FQKGAUHPW.png)

成功！

_注：pyc文件一定要有后缀名pyc，不然会报错_

## 二、PYZ文件的加密问题

有些时候在**步骤1 exe→pyc**的过程中，会出现PYZ中的文件无法正常提取（archive_viewer.py），或者提取出来后显示encrypted（pyinstxtractor.py）的问题。

这个问题可以使用**参考文章2和3**中的方法解决：

PYZ文件加密的密钥保存在`pyimod00_crypto_key`文件中，该文件也是一个pyc文件，可以使用上面介绍的方法进行反编译，然后就可以获得密钥：

![](https://bbs.kanxue.com/upload/attach/202012/600394_8E82CZ6YCU6BT8P.png)

之后的解密脚本有三个可供选择，均在**参考文章3**中，根据pyinstaller的版本不同选择不同的脚本，使用时需要替换其中的key、header以及待解密文件名和目标文件名，执行后即可获得解密后的pyc文件，再使用uncompyle6反编译即可。

_注：这三个脚本中，第一个脚本适用于PyInstaller<4.0，使用python2执行；第二个和第三个脚本适用于PyInstaller≥4.0，使用python3执行。_

![](https://bbs.kanxue.com/upload/attach/202012/600394_M8F4F4MBCMHKS2V.png)

## 参考文章

1.  [反编译python打包的exe文件](https://www.cnblogs.com/QKSword/p/10540431.html)
    
2.  [Extracting encrypted pyinstaller executables](https://0xec.blogspot.com/2017/02/extracting-encrypted-pyinstaller.html)
    
3.  [pyinstxtractor wiki](https://github.com/extremecoders-re/pyinstxtractor/wiki/Frequently-Asked-Questions)
    
4.  [https://fishc.com.cn/forum.php?mod=viewthread&tid=131172&page=1#pid3772102](https://fishc.com.cn/forum.php?mod=viewthread&tid=131172&page=1#pid3772102)
    

  

[IDA 特训营](https://www.kanxue.com/book-section_list-156.htm)

最后于 2020-12-15 15:01 被LarryS编辑 ，原因： 增加原创标志以及调试逆向分类

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
