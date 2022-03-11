# BkP-CTF 2013 MITM - xelz's blog
前两天 BkP 的 CTF 练习赛中的一道题，crypto 200，题目如下

> message 1: QUVTLTI1NiBFQ0IgbW9kZSB0d2ljZSwgdHdvIGtleXM=
>
> encrypted: THbpB4bE82Rq35khemTQ10ntxZ8sf7s2WK8ErwcdDEc=
>
> message 2: RWFjaCBrZXkgemVybyB1bnRpbCBsYXN0IDI0IGJpdHM=
>
> encrypted: 01YZbSrta2N+1pOeQppmPETzoT/Yqb816yGlyceuEOE=
>
> ciphertext: s5hd0ThTkv1U44r9aRyUhaX5qJe561MZ16071nlvM9U=

看到最后的等号首先就想到了 base64 编码，decode 之后得到

> message1: AES-256 ECB mode twice, two keys
>
> message2: Each key zero until last 24 bits
>
> 两轮 AES-256 加密，padding=ECB，key 不一样，但是前面都是 0x00，只有最后 24 位需要破解
>
> 密文都是 2 进制不可读，不贴了

题目提示了是 256 位 (32 字节的 key)，前 29 个字节都是 0，需要破解两个 key 的后 3 个字节，纯暴力方式需要尝试 224 \* 224 = 248 ≈ 2.81e14 种可能，这么大的计算量，显然是不现实的。

暴力破解，估计要用到 hadoop 集群了。

其实，当时忽略了一个细节，就是题目：MITM，google 一下出来的都是 Man-in-the-middle Attack（中间人攻击），似乎跟这个题目半毛钱关系都没有，换用 wikipedia 得到了我们想要的东西：

-   Man-in-the-middle attack, a computer networking attack method
-   Meet-in-the-middle attack, a cryptographic attack method

很显然，Meet-in-the-middle attack 应该就是我们想找的东西了

> Assume the attacker knows a set of plaintext P and ciphertext C that satisfies the following:
>
> -   C=ENCk2(ENCk1(P))
> -   P=DECk1(DECk2©
>
> where ENC is the encryption function, DEC the decryption function defined as ENC-1 (inverse mapping) and k1 and k2 are two keys.
>
> The attacker can then compute ENCk1(P) for all possible keys k1. Afterwards he can decrypt the ciphertext by computing DECk2© for each k2. Any matches between these two resulting sets are likely to reveal the correct keys. (To speed up the comparison, the ENCk1(P) set can be stored in an in-memory lookup table, then each DECk2© can be matched against the values in the lookup table to find the candidate keys)

这个模型跟题目所设的是完全一样的，思路给的很清楚了，先穷举 key1，计算出明文经过所有可能的 key1 加密后的结果，将结果存于内存中，然后穷举 key2，计算密文经过 key2 解密后的结果，与内存中的结果集进行比对（因为 AES 是对称加密，加密跟解密是用的相同的 key），如果有一致的，就表明破解成功了，这样算起来，时间复杂度只有 224 + 224 = 225

```
#!/usr/bin/env python

#! -*- coding: utf-8 -*-



from Crypto.Cipher import AES

from base64 import b64decode



def aes_encrypt(key, text, mode=AES.MODE_ECB):

    encryptor = AES.new(key, mode)

    ciphertext = encryptor.encrypt(text)

    return ciphertext



def aes_decrypt(key, text, mode=AES.MODE_ECB):

    decryptor = AES.new(key, mode)

    plaintext = decryptor.decrypt(text)

    return plaintext



if __name__ == '__main__':

    message1 = b64decode('QUVTLTI1NiBFQ0IgbW9kZSB0d2ljZSwgdHdvIGtleXM=')     # 'AES-256 ECB mode twice, two keys'

    ciphertext1 = b64decode('THbpB4bE82Rq35khemTQ10ntxZ8sf7s2WK8ErwcdDEc=')  # '\x4c\x76\xe9\x07\x86\xc4\xf3\x64\x6a\xdf\x99\x21\x7a\x64\xd0\xd7\x49\xed\xc5\x9f\x2c\x7f\xbb\x36\x58\xaf\x04\xaf\x07\x1d\x0c\x47'

    # message2 = b64decode('RWFjaCBrZXkgemVybyB1bnRpbCBsYXN0IDI0IGJpdHM=')     # 'Each key zero until last 24 bits'

    # ciphertext2 = b64decode('01YZbSrta2N+1pOeQppmPETzoT/Yqb816yGlyceuEOE=')  # '\xd3\x56\x19\x6d\x2a\xed\x6b\x63\x7e\xd6\x93\x9e\x42\x9a\x66\x3c\x44\xf3\xa1\x3f\xd8\xa9\xbf\x35\xeb\x21\xa5\xc9\xc7\xae\x10\xe1'

    ciphertext = b64decode('s5hd0ThTkv1U44r9aRyUhaX5qJe561MZ16071nlvM9U=')   # '\xb3\x98\x5d\xd1\x38\x53\x92\xfd\x54\xe3\x8a\xfd\x69\x1c\x94\x85\xa5\xf9\xa8\x97\xb9\xeb\x53\x19\xd7\xad\x3b\xd6\x79\x6f\x33\xd5'



    prefix = '\0' * 29

    clist = range(256)

    mitms = []

    for a in clist:

        for b in clist:

            for c in clist:

                key1 = prefix + chr(a) + chr(b) + chr(c)

                mitm1 = aes_encrypt(key1, message1)

                mitms.append(mitm1)

    mitms_set = set(mitms)  # convert to set for faster index

    for a in clist:

        for b in clist:

            for c in clist:

                key2 = prefix + chr(a) + chr(b) + chr(c)

                anmitm1 = aes_decrypt(key2, ciphertext1)

                if anmitm1 in mitms_set:

                    key1_suffix = mitms.index(anmitm1)

                    print 'key1: %s' % repr(key1)

                    print 'key2: %s' % repr(key2)

                    msg = aes_decrypt(key1, aes_decrypt(key2, ciphertext))

                    print 'message is: %s' % msg

```

大概 5 分钟左右就跑完了，缓存 key1 的加密结果用了 1.65G 内存，如果内存不够，可以对 key1 分段跑，不过时间就要相应变长。

key1:

> \\x9a\\xe8\\x07

key2:

> \\xff?E

message is:

> This time I didn’t include sol’n 
>  [https://xelz.info/blog/2013/06/12/bkp-ctf-2013-mitm/](https://xelz.info/blog/2013/06/12/bkp-ctf-2013-mitm/)
