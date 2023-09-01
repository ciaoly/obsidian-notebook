
# 二进制相关操作

#### 运算符:

| 运算符 | 描述     |
| :----: | :------- |
| -band  | 二进制与 |
|  -bor  | 二进制或 |
| -bnot  | 二进制非 |

## 二进制数组

### 创建数组

#### Creating an array from a list of numerical literals

An [array](https://renenyffenegger.ch/notes/Windows/PowerShell/language/type/array/index) of bytes can simply be created by assigning an array of numbers (technically: bytes) to a typed [variable](https://renenyffenegger.ch/notes/Windows/PowerShell/language/variable/index):

```powershell
[byte[]] $byte_array = 65,66,67,90
```
Of course, typically, such numbers come in hexadecimal form in which case they need to be prepended with `0x`:

```powershell
[byte[]] $another_array = 0x41, 0x42, 0x43, 0x5A
```

If the array of hexadecimal number comes from a source where it is not already prefixed and manually prefixing them is too tedious, a [pipeline](https://renenyffenegger.ch/notes/Windows/PowerShell/pipeline/index) helps to convert the hexadecimal numbers to a byte:

```powershell
[byte[]] $xyz = '41', '42', '43', '5A' | foreach-object { invoke-expression "0x$_" }
```

Because some of the values of the last example contain characters, these values had to be enclosed in quotes.

To make things easier, we can put such values into a string and create an array using the [`-split` operator](https://renenyffenegger.ch/notes/Windows/PowerShell/language/operator/string-manipulation/index#ps-split-join):

```powershell
[byte[]] $abc = "41 42 43 5A" -split ' ' | foreach-object { invoke-expression "0x$_" }
```

#### Create an array of n elements, each initialized to 0

The following constructs create `byte` arrays, each of which has 10 elements and whose values are initialized to `0`.

```powershell
$array_1 = [byte[]]::new(10)
$array_2 = new-object byte[] 10
$array_3 = [System.Array]::CreateInstance([byte], 10)
```

#### Creating higher dimension arrays

The following constructs create arrays of higher dimensions:
```powershell
[byte[,]] $four_by_eight = [System.Array]::CreateInstance([byte], @(4, 8))
[byte[,,,]] $b_3x4x5x6 = [byte[,,,]]::new(3,4,5,6)
```

### Byte数组与Hex字符串之间相互转换

```powershell
$str = '11aabbccdd';
$bytes = @();

$bytes = [int[]] -split ($str -replace "(..)",'0x$1 ');
$str = [char[]]$bytes -join ""

```

### 字符串直接转Byte数组

```powershell
[System.Text.Encoding]::UTF8.GetBytes("ghijklmn")
```

### Base64字符串转Byte数组
[[Powershell常见编码或加密#base64]]

### 数字与Byte数组转换

参考链接: [BitConverter Class (System) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter?view=net-7.0)

相关转换函数如下:

|Type|To byte conversion|From byte conversion|
|---|---|---|
|[Boolean](https://learn.microsoft.com/en-us/dotnet/api/system.boolean?view=net-7.0)|[GetBytes(Boolean)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-boolean))|[ToBoolean](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.toboolean?view=net-7.0)|
|[Char](https://learn.microsoft.com/en-us/dotnet/api/system.char?view=net-7.0)|[GetBytes(Char)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-char))|[ToChar](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.tochar?view=net-7.0)|
|[Double](https://learn.microsoft.com/en-us/dotnet/api/system.double?view=net-7.0)|[GetBytes(Double)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-double))  <br> -or-  <br>[DoubleToInt64Bits(Double)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.doubletoint64bits?view=net-7.0#system-bitconverter-doubletoint64bits(system-double))|[ToDouble](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.todouble?view=net-7.0)   <br>-or-  <br> [Int64BitsToDouble](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.int64bitstodouble?view=net-7.0)|
|[Int16](https://learn.microsoft.com/en-us/dotnet/api/system.int16?view=net-7.0)|[GetBytes(Int16)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-int16))|[ToInt16](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.toint16?view=net-7.0)|
|[Int32](https://learn.microsoft.com/en-us/dotnet/api/system.int32?view=net-7.0)|[GetBytes(Int32)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-int32))|[ToInt32](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.toint32?view=net-7.0)|
|[Int64](https://learn.microsoft.com/en-us/dotnet/api/system.int64?view=net-7.0)|[GetBytes(Int64)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-int64))|[ToInt64](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.toint64?view=net-7.0)|
|[Single](https://learn.microsoft.com/en-us/dotnet/api/system.single?view=net-7.0)|[GetBytes(Single)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-single))|[ToSingle](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.tosingle?view=net-7.0)|
|[UInt16](https://learn.microsoft.com/en-us/dotnet/api/system.uint16?view=net-7.0)|[GetBytes(UInt16)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-uint16))|[ToUInt16](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.touint16?view=net-7.0)|
|[UInt32](https://learn.microsoft.com/en-us/dotnet/api/system.uint32?view=net-7.0)|[GetBytes(UInt32)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-uint32))|[ToUInt32](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.touint32?view=net-7.0)|
|[UInt64](https://learn.microsoft.com/en-us/dotnet/api/system.uint64?view=net-7.0)|[GetBytes(UInt64)](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.getbytes?view=net-7.0#system-bitconverter-getbytes(system-uint64))|[ToUInt64](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter.touint64?view=net-7.0)|

其中, C#的`Char`是Unicode字符, 占两个字节. 转换时按照机器的大小端序列开展, 如果需要"网络序", 则首先需要使用使用 `[System.BitConverter]::IsLittleEndian` 判断机器是否为小端序(一般都是小端序), 再使用 `[Array]::Reverse()` 转换字节顺序, 例如:

```powershell
#读取前四个字节获取数据包的大小
$stream.Read($buffer, 0, 4);
if ([System.BitConverter]::IsLittleEndian) {
	[System.Array]::Reverse($buffer);
}
[System.BitConverter]::ToUInt32($barr, 0)
```

## 读取二进制文件

### 一次性读取
```powershell
[System.IO.File]::ReadAllBytes('c:\test.log');
```

### 切片读取(大文件)

```powershell
$chunksize = 1024  
$stream = [System.IO.File]::OpenRead($inFile)
$chunkNum = 1
[Int64]$curOffset = 0 
$buffer = [byte[]]::New($chunksize)
while( $bytesRead = $stream.Read($buffer,0,$chunksize)){  
    echo "Readed: $($bytesRead)";
    # Some-Func -Input $buffer[0..$bytesRead]
    $chunkNum += 1  
    $curOffset += $bytesRead  
    $stream.seek($curOffset,0);
}
```

### 文件切片

参考链接: [FileStream Class (System.IO) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.io.filestream?view=net-7.0)
```powershell
$chunksize = 1024
$stream = [System.IO.File]::OpenRead($inFile)  
$chunkNum = 1  
[Int64]$curOffset = 0  
$barr = New-Object byte[] $chunksize  
while( $bytesRead = $stream.Read($barr,0,$chunksize)){  
  $outFile ="$outPrefix$chunkNum"  
  $ostream = [System.IO.File]::OpenWrite($outFile)  
  $ostream.Write($barr,0,$bytesRead);  
  $ostream.close();  
  echo "wrote $outFile"  
  $chunkNum += 1  
  $curOffset += $bytesRead  
  $stream.seek($curOffset,0);
}
```
