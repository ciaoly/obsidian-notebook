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


