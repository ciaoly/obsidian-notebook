
# 函数

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

### 参数

#### 位置参数 $args



### 关于Param()

> The assignment expression is not valid. The input to an assignment operator must be an object that is able to accept assignments, such as a variable or a property.

当出现上述错误时, 说明`Param()`位置不对. 

> Params should be at the start of the file, or at the start of a function.


-------

* [PowerShell 函数 - sparkdev - 博客园 (cnblogs.com)](https://www.cnblogs.com/sparkdev/p/8242167.html)
