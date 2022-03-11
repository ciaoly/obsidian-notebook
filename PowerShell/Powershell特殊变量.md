# 特殊变量

  PowerShell的特殊变量由系统自动创建。用户自定义的变量名称应该不和特殊变量相同。

|      变量       | 描述                                                    |
| :-------------: | ------------------------------------------------------- |
|      `$^`       | 前一命令行的第一个标记                                  |
|      `$$`       | 前一命令行的最后一个标记                                |
|      `$_`       | 表示表示当前循环的迭代变量。                            |
|      `$?`       | 前一命令执行状态，成功（`$Ture`） 或者 失败（`$False`） |
|     `$Args`     | 为脚本或者函数指定的参数                                |
|    `$Error`     | 错误发生时，错误对象存储于变量 `$Error` 中              |
|   `$Foreach`    | 引用`foreach`循环中的枚举器                             |
|     `$Home`     | 用户的主目录                                            |
|     `$Host`     | 引用宿主 POWERSHELL 语言的应用程序                      |
|    `$Input`     | 通过管道传递给脚本的对象的枚举器                        |
| `$LastExitCode` | 上一程序或脚本的退出代码                                |
|   `$Matches`    | 使用 `–match` 运算符找到的匹配项的哈希表                |
|    `$PSHome`    | Windows PowerShell 的安装位置                           |
|   `$profile`    | 标准配置文件（可能不存在）                              |
|  `$StackTrace`  | Windows PowerShell 捕获的上一异常                       |
|    `$Switch`    | `switch` 语句中的枚举器                                 |


-------

参考:

* [PowerShell随笔2_分支 选择 循环 特殊变量 - momingliu11 - 博客园 (cnblogs.com)](https://www.cnblogs.com/dreamer-fish/p/3724217.html)