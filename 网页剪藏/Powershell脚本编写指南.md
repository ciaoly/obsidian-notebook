## 程序控制结构

### 条件

#### 运算符:

| C++  | Powershell | English       |
| ---- | ---------- | ------------- |
| >    | -gt        | greater than  |
| <    | -lt        | less than     |
| ==   | -eq        | equal         |
| >=   | -ge        | greater equal |
| <=   | -le        | less equal    |
| !=   | -ne        | not equal     |
| &&   | -and       | and           |
| \|\| | -or        | or            |
| !    | -not       | not           |

```powershell
if($a -eq 4) {
    #...
} elseif(($a -gt 5) -and ($a -lt 6)){
    #...
} else {
    #...
}
```

### 循环

```powershell
foreach($item in $arr) {
    #...$item是元素不是索引
}

for($i=0; $i -lt $arr.Count; $i++){
    #...
}

$i=0
while($i -lt 10){
    #...break;
    $i++
}

do {
   #...
} until($n -gt 3)

do {
   #...continue;
} while($n -lt 3)
```

### switch

```powershell
# switch [-regex | -wildcard | -exact] [-casesensitive]（表达式）| -file filename   #表达式可以为数组，为数组时顺序处理数组每一项 
switch($a){
    {$a -eq 1} { ... }
    {$a -eq 2} { ... break;}
    {$a -eq 3} { ... break;}
    default {...}
}

$b="Hello"
switch($b){
    "Hello" { ... }
    "Hi" { ... }
    "HeHe" { ... }
    default {...}
}
```

### try..catch...

```powershell
# try {<statement list>}
# catch [[<error type>][',' <error type>]*] {<statement list>}
# finally {<statement list>}

try {
   $wc = new-object System.Net.WebClient
   $wc.DownloadFile("http://www.contoso.com/MyDoc.doc","c:\temp\MyDoc.doc")
}
catch [System.Net.WebException],[System.IO.IOException] {
    "Unable to download MyDoc.doc from http://www.contoso.com."
}
catch {
# Within a Catch block, the current error can be accessed using $_, which is also known as $PSItem. The object is of type ErrorRecord.
  Write-Host "An error occurred:"
  Write-Host $_
  Write-Host $_.ScriptStackTrace
}
```

> Within a `Catch` block, the current error can be accessed using `$_`, which is also known as `$PSItem`. The object is of type **ErrorRecord**.

#### 抛出异常

```powershell
#Throw  字符串|异常|ErrorRecord
```



## 特殊变量

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



## 函数

### 变量作用域

```powershell
#一如下：（函数中改变变量值并不影响实际值）
$var1=10
function one{"The Variable is $var1"}
function two{$var1=20;one}
one
two
one
#执行结果：
#             The Variable is 10
#             The Variable is 20
#             The Variable is 10

#用法二如下：（函数中变量值的改变要用$Script:var的形式）
$var1=10
function one{"The Variable is $var1"}
function two{$Script:var1=20;one}
one
two
one
#执行结果：
#              The Variable is 10
#              The Variable is 20
#              The Variable is 20
```



## 字符串相关操作

#### 运算符:

|  运算符  | 描述                 | 备注                                                         |
| :------: | -------------------- | :----------------------------------------------------------- |
|    \+    | 连接两个字符串       |                                                              |
|    \*    | 按指定次数重复字符串 |                                                              |
|    -f    | 设置字符串格式       |                                                              |
| -replace | 替换运算符           | 用法："abcd" -replace "bc","TEST"  返回结果：aTESTd; 支持正则表达式 |
|  -match  | 正则表达式匹配       |                                                              |
|  -like   | 通配符匹配           |                                                              |



## 二进制相关操作

#### 运算符:

| 运算符 | 描述     |
| :----: | :------- |
| -band  | 二进制与 |
|  -bor  | 二进制或 |
| -bnot  | 二进制非 |

### Byte数组与Hex字符串之间相互转换

```powershell
```



## 数组相关操作

### 逻辑运算符应用于数组(类似于Matlab和Python)

```powershell
#此数组中是否包含3：
  1,2,3,5,3,2 –contains 3
#返回所有等于3的元素：
  1,2,3,5,3,2 –eq 3
#返回所有小于3的元素：
  1,2,3,5,3,2 –lt 3
#测试 2 是否存在于集合中：
  if (1..5 –contains 2)
```

### 元素操作

```powershell
# 追加元素
$arr += "a";
```



## 网络相关操作



-------

参考:

* [PowerShell随笔2_分支 选择 循环 特殊变量 - momingliu11 - 博客园 (cnblogs.com)](https://www.cnblogs.com/dreamer-fish/p/3724217.html)
* [PowerShell 函数 - sparkdev - 博客园 (cnblogs.com)](https://www.cnblogs.com/sparkdev/p/8242167.html)

