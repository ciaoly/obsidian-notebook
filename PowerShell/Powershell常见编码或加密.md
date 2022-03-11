## base64

文本和base64互转需要经过bytes中继,因此涉及到编码问题.

```powershell
#转base64
$Text = 'ouser:v3$34@#85b&g%fD79a3nf'
$Bytes = [System.Text.Encoding]::Unicode.GetBytes($Text)
$EncodedText =[Convert]::ToBase64String($Bytes)

#base64转
$EncodedText = "VABoAGkAcwAgAGkAcwAgAGEAIABzAGUAYwByAGUAdAAgAGEAbgBkACAAcwBoAG8AdQBsAGQAIABiAGUAIABoAGkAZABlAG4A"
#转换结果为utf16编码:
$DecodedText = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($EncodedText))
#转换结果为utf8编码
$DecodedText = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($EncodedText))
#转换结果为ASCII
$DecodedText = [System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String($EncodedText))
```

## md5

```powershell
#字符串:
$someString = "Hello, World!"
$md5 = New-Object -TypeName System.Security.Cryptography.MD5CryptoServiceProvider
$utf8 = New-Object -TypeName System.Text.UTF8Encoding
$hash = [System.BitConverter]::ToString($md5.ComputeHash($utf8.GetBytes($someString)))

#文件
Get-FileHash -Algorithm MD5 path\to\file
```

