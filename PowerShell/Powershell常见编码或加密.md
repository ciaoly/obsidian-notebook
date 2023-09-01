## 编码

### base64

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

### URL编码
`[System.Web.HttpUtility]::UrlEncode()`

## 密码学相关

不管是下述的哈希还是加密, 在C#里面都可以通过`Cryptography`类实现, 具体可参考: [System.Security.Cryptography Namespace | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography?view=net-7.0)

## 哈希

### md5

```powershell
#字符串:
$someString = "Hello, World!"
$md5 = New-Object -TypeName System.Security.Cryptography.MD5CryptoServiceProvider
$utf8 = New-Object -TypeName System.Text.UTF8Encoding
$hash = [System.BitConverter]::ToString($md5.ComputeHash($utf8.GetBytes($someString)))

#文件
Get-FileHash -Algorithm MD5 path\to\file
```

## 加密

### AES

```powershell
function Create-AesManagedObject($key, $IV, $mode) {
    $aesManaged = New-Object "System.Security.Cryptography.AesManaged"

    if ($mode="CBC") { $aesManaged.Mode = [System.Security.Cryptography.CipherMode]::CBC}
    elseif ($mode="CFB") {$aesManaged.Mode = [System.Security.Cryptography.CipherMode]::CFB}
    elseif ($mode="CTS") {$aesManaged.Mode = [System.Security.Cryptography.CipherMode]::CTS}
    elseif ($mode="ECB") {$aesManaged.Mode = [System.Security.Cryptography.CipherMode]::ECB}
    elseif ($mode="OFB"){$aesManaged.Mode = [System.Security.Cryptography.CipherMode]::OFB}


    $aesManaged.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7
    $aesManaged.BlockSize = 128
    $aesManaged.KeySize = 256
    if ($IV) {
        if ($IV.getType().Name -eq "String") {
            $aesManaged.IV = [System.Convert]::FromBase64String($IV)
        }
        else {
            $aesManaged.IV = $IV
        }
    }
    if ($key) {
        if ($key.getType().Name -eq "String") {
            $aesManaged.Key = [System.Convert]::FromBase64String($key)
        }
        else {
            $aesManaged.Key = $key
        }
    }
    $aesManaged
}

function Encrypt-String($key, $plaintext) {
    $bytes = [System.Text.Encoding]::UTF8.GetBytes($plaintext)
    $aesManaged = Create-AesManagedObject $key
    $encryptor = $aesManaged.CreateEncryptor()
    $encryptedData = $encryptor.TransformFinalBlock($bytes, 0, $bytes.Length);
    [byte[]] $fullData = $aesManaged.IV + $encryptedData
    [System.Convert]::ToBase64String($fullData)
}

function Decrypt-String($key, $encryptedStringWithIV) {
    $bytes = [System.Convert]::FromBase64String($encryptedStringWithIV)
    $IV = $bytes[0..15]
    $aesManaged = Create-AesManagedObject $key $IV
    $decryptor = $aesManaged.CreateDecryptor();
    $unencryptedData = $decryptor.TransformFinalBlock($bytes, 16, $bytes.Length - 16);
    $aesManaged.Dispose()
    [System.Text.Encoding]::UTF8.GetString($unencryptedData).Trim([char]0)
}

function Create-AesKey() {
    $aesManaged = Create-AesManagedObject
    $aesManaged.GenerateKey()
    [System.Convert]::ToBase64String($aesManaged.Key)
}

```

### DES

```powershell
$plaintext="Hello"
$key="0001020304050607"
$iv="0001020304050607"

$plaintext=$Args[0]
$key=$Args[1]
$iv=$Args[2]
$mode=$Args[3]

$des=[System.Security.Cryptography.Des]::Create()

if ($mode -eq "CBC") { $des.Mode = [System.Security.Cryptography.CipherMode]::CBC }
elseif ($mode -eq "ECB") {$des.Mode = [System.Security.Cryptography.CipherMode]::ECB}



$des.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7
$des.BlockSize = 64
$des.IV=[System.Convert]::FromHexString($iv)
$des.Key = [System.Convert]::FromHexString($key)


$bytes = [System.Text.Encoding]::UTF8.GetBytes($plaintext)
$encryptor = $des.CreateEncryptor()
$encryptedData = $encryptor.TransformFinalBlock($bytes, 0, $bytes.Length);

$decryptor = $des.CreateDecryptor()
$pl = $decryptor.TransformFinalBlock($encryptedData, 0, $encryptedData.Length);

"== DES Encryption == "
"Message: " + $plaintext
"Key: " + $key
"IV: " + $iv
"Mode: " + $mode
"Encrypted Data: " + [System.Convert]::ToHexString($encryptedData)
"Decrypted Data: " + [System.Text.Encoding]::ASCII.GetString($pl)
```

### 加密模式

[参考: ](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.ciphermode?view=net-7.0#fields)

|Mode |value | info |
|---|---|---|
|CBC|1|The Cipher Block Chaining (`CBC`) mode introduces feedback. Before each plain text block is encrypted, it is combined with the cipher text of the previous block by a bitwise exclusive OR operation. This ensures that even if the plain text contains many identical blocks, they will each encrypt to a different cipher text block. The initialization vector is combined with the first plain text block by a bitwise exclusive OR operation before the block is encrypted. If a single bit of the cipher text block is mangled, the corresponding plain text block will also be mangled. In addition, a bit in the subsequent block, in the same position as the original mangled bit, will be mangled.|
|CFB|4|The Cipher Feedback (`CFB`) mode processes small increments of plain text into cipher text, instead of processing an entire block at a time. This mode uses a shift register that is one block in length and is divided into sections. For example, if the block size is 8 bytes, with one byte processed at a time, the shift register is divided into eight sections. If a bit in the cipher text is mangled, one plain text bit is mangled and the shift register is corrupted. This results in the next several plain text increments being mangled until the bad bit is shifted out of the shift register. The default feedback size can vary by algorithm, but is typically either 8 bits or the number of bits of the block size. You can alter the number of feedback bits by using the [FeedbackSize](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.symmetricalgorithm.feedbacksize?view=net-7.0#system-security-cryptography-symmetricalgorithm-feedbacksize) property. Algorithms that support CFB use this property to set the feedback.|
|CTS|5|The Cipher Text Stealing (`CTS`) mode handles any length of plain text and produces cipher text whose length matches the plain text length. This mode behaves like the `CBC` mode for all but the last two blocks of the plain text.|
|ECB|2|The Electronic Codebook (`ECB`) mode encrypts each block individually. Any blocks of plain text that are identical and in the same message, or that are in a different message encrypted with the same key, will be transformed into identical cipher text blocks. **Important**: This mode is not recommended because it opens the door for multiple security exploits. If the plain text to be encrypted contains substantial repetition, it is feasible for the cipher text to be broken one block at a time. It is also possible to use block analysis to determine the encryption key. Also, an active adversary can substitute and exchange individual blocks without detection, which allows blocks to be saved and inserted into the stream at other points without detection.|
|OFB|3|The Output Feedback (`OFB`) mode processes small increments of plain text into cipher text instead of processing an entire block at a time. This mode is similar to `CFB`; the only difference between the two modes is the way that the shift register is filled. If a bit in the cipher text is mangled, the corresponding bit of plain text will be mangled. However, if there are extra or missing bits from the cipher text, the plain text will be mangled from that point on.|

### 填充模式

[PaddingMode Enum (System.Security.Cryptography) | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography.paddingmode?view=net-7.0#fields)

|方式| 值 |说明|
|---|---|---|
|ANSIX923|4|The ANSIX923 padding string consists of a sequence of bytes filled with zeros before the length.|
|ISO10126|5|The ISO10126 padding string consists of random data before the length.|
|None|1|No padding is done.|
|PKCS7|2|The PKCS #7 padding string consists of a sequence of bytes, each of which is equal to the total number of padding bytes added.|
|Zeros|3|The padding string consists of bytes set to zero.|

