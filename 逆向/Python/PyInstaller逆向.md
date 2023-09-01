PyInstaller打包的软件有一个很常见的特征, 就是它的icon是一个磁盘上趴了一条蛇的图标: ![](https://avatars.githubusercontent.com/u/1215332?s=200&v=4)
当然这个icon是可以修改的, 更正确的方法是看看它的依赖库, 更简单的是使用*EXEInfoPe*看一看PE文件的信息, 如果出现*PyInstaller*字样基本上就是PyInstall打包的EXE了.

关于这方面逆向, 可以参考: [[【原创】python exe文件解包方法总结]]
