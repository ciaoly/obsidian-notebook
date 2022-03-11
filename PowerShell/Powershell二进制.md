
# 二进制相关操作

#### 运算符:

| 运算符 | 描述     |
| :----: | :------- |
| -band  | 二进制与 |
|  -bor  | 二进制或 |
| -bnot  | 二进制非 |

### Byte数组与Hex字符串之间相互转换

```powershell
$str = '11aabbccdd';
$bytes = @();

$bytes = [int[]] -split ($str -replace "(..)",'0x$1 ');
$str = [char[]]$bytes -join ""

```


