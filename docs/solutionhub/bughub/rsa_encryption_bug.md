---
title: 多语言之间的RSA加解密问题
date: 2022-07-28 21:46:56
tags: 
  - Bug
---

# 多语言之间的RSA加解密问题

## 事出有因

在完成创建数据源的需求时，需要对数据源的用户密码等相关敏感信息进行加密传递，Vue前端加密，Java后端和Python后端解密。经过方案调研后，最终确定采用RSA加解密的方案。  

## Bug现场

由Java后端通过工具生成公私钥字符串，并将公钥给到前端，前端在创建数据源利用公钥对用户输入的敏感信息进行加密，Java服务接收请求进行校验，并保存。Python服务需要在使用数据源时进行敏感信息解密，此时遇到的问题是，经java服务解密校验通过，而Python则无法进行解密。

## 迎刃而解

根因分析：

经过对Java服务和Python服务的加j解密实现进行深入研究，发现Java中使用的PKCS8,而Python中只有PKCS1，因为此协议不匹配，导致无法解密。

> PKCS #1为OpenSSH默认使用的加密标准 [^2]:PKCS #8为Java默认使用的加密标准

方案1尝试：试图在python的工具包中找到关于PKCS8的实现，未果。

方案2尝试：试图在Java的工具包中找到关于PKCS1的实现，未果。

方案3尝试：经过一番搜索，找到一种思路，将PKCS8生成的私钥转换成PKCS1的私钥，再进行尝试，成功。

完成流程总结：

-   使用openssl生成秘钥

```bash
openssl genrsa -out private.pem 1024
```

-   查看私钥

```bash
-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQDAQWZDPB5q3B0hKNO8Schw4ihwrJAITWHMMImjSKcRUSbAaKsM
3kLhI8Sx7mPHVW21W6X4MOWyPLAlAOfX3DWNSHBThsJKOEeIlNmZ68MjCE6hWx+W
fpUtPWmj8CBNUuZrHmAip4/2TTMHEfJPPzRA5FBmzjH7cn/5w+QlZjeINwIDAQAB
AoGAR4ILv0ZFKgnk68h7uLTY0OPNltsYV34wufnzt7/2JALDHx3PQWIKDiN3rZa1
lha4T5RfDwlg5gKcoabMlQVbBSPtvr2Rw0v9z8Ib7fsPqDJWUAkFL24DijpVEHiV
*******************
w7ZTo+aFijPS3ngRL6ksmIM+gbMwFQBBgnlKgmSrsA7wjpql9A+LLLyLoxAg+hFf
5HgxInkJAkEAyP4R3tT2i6s/ZT01RPhNYJaHlaRsHAluPlyRDKCf/TLnDTHfB141
kflUi/kcOMrSnwBYCDn9zh0qpf4O26iHPwJAHrOxcvbILmIhPHQucxQ0vf+S+dhs
43K/WlkQ85BdgCTR8GTEgmQQ7TnY1a57k10jacbFKRjb19aiwQwrSbXRyQJAIVro
pE47TBpzPt3VhUifrrPMdG0A8/YssXSzLaFKa76S0YHBPXvI8Byshz4kDNi7818f
PN5C3H3CoiFzXciuCQJAbLm5SfxCony3z5jkkQRM8Agl7sPCEt095zVJX+ln7apv
t9Ci4v8kwGLz8un7PmLOvmMh7GzSh6qx7PinpI1WRw==
-----END RSA PRIVATE KEY-----
```

-   生成查看公钥

```bash
openssl rsa -in private.pem -out public.pem -pubout

-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDAQWZDPB5q3B0hKNO8Schw4ihw
rJAITWHMMImjSKcRUSbAaKsM3kLhI8Sx7mPHVW21W6X4MOWyPLAlAOfX3DWNSHBT
*******************
zjH7cn/5w+QlZjeINwIDAQAB
-----END PUBLIC KEY-----
```

-   将PKCS1的私钥转换为PKCS8

```bash
openssl pkcs8 -topk8 -inform PEM -in private.pem -outform pem -nocrypt -out pkcs8.pem
```

-   配置到代码中，即可

```
-----BEGIN PRIVATE KEY-----
MIICdQIBADANBgkqhkiG9w0BAQEFAASCAl8wggJbAgEAAoGBAMBBZkM8HmrcHSEo
07xJyHDiKHCskAhNYcwwiaNIpxFRJsBoqwzeQuEjxLHuY8dVbbVbpfgw5bI8sCUA
59fcNY1IcFOGwko4R4iU2ZnrwyMITqFbH5Z+lS09aaPwIE1S5mseYCKnj/ZNMwcR
8k8/NEDkUGbOMftyf/nD5CVmN4g3AgMBAAECgYBHggu/RkUqCeTryHu4tNjQ482W
2xhXfjC5+fO3v/YkAsMfHc9BYgoOI3etlrWWFrhPlF8PCWDmApyhpsyVBVsFI+2+
vZHDS/3Pwhvt+w+oMlZQCQUvbgOKOlUQeJXezM9Ax6VmBKjM3RqctZ6IPphpO2KX
2wprrUzwBhQkCa7MgQJBAPTfMtWLBI3DvAnDtlOj5oWKM9LeeBEvqSyYgz6BszAV
AEGCeUqCZKuwDvCOmqX0D4ssvIujECD6EV/keDEieQkCQQDI/hHe1PaLqz9lPTVE
+E1gloeVpGwcCW4+XJEMoJ/9MucNMd8HXjWR+VSL+Rw4ytKfAFgIOf3OHSql/g7b

*************

rnuTXSNpxsUpGNvX1qLBDCtJtdHJAkAhWuikTjtMGnM+3dWFSJ+us8x0bQDz9iyx
dLMtoUprvpLRgcE9e8jwHKyHPiQM2LvzXx883kLcfcKiIXNdyK4JAkBsublJ/EKi
fLfPmOSRBEzwCCXuw8IS3T3nNUlf6Wftqm+30KLi/yTAYvPy6fs+Ys6+YyHsbNKH
qrHs+KekjVZH
-----END PRIVATE KEY-----
```

## 扩展

### Python代码片段

```python
import base64
# pip install pycryptodome
from Crypto.Cipher import PKCS1_v1_5
from Crypto.PublicKey import RSA

PUBLIC_KEY = "MRFM4nWu25GsG+gLwkqClZk1DUva\nKfkKK73ZQ8mcLI3Pbx8zaX+1P38S7KUIOK8QI9gzsIZ0RMamOmrwl4ECAwEAAQ=="
PRIVATE_KEY = "MIIBOQIBAA8mcLI3Pbx8z\naX+1P38S7KUIOK8QI9gzsIZ0RMamOmrwl4ECAwEAAQJAdZzQARRCEZ1P9GkIVbVa\naYWcXSe53Paet+YeteIN3xJELq4qYYuYAGlF2HmP9wpKZHXHdmlGEs8fi+pUMEGP\n8QIhAOYVlA5Jihog5jSEb5S2Id7Z7LjrusLs67MtjjQzj3ddAiEAm1QpjN/QzB4l\nRxIHG2BlzwmU/vSJYGH8ePkRMuC8knUCIHF+43HIxN7uq5/sVD4/OaX8SdFONupA\nhGP2bNdDN9nhAiATKYXauD3VAJ8Orn2r9e95ZDA6Z8aO2mfAMNHbWfhJhQIgCtFH\nzTtqqn/mgfftb/9g3vYyBRAz90vlKAaHgd9/U/k="


def decrypt(data: str, private_key: str) -> str:
    convert_data = base64.b64decode(bytes(data, encoding="utf8"))
    rsa_key = RSA.importKey(base64.b64decode(private_key))
    cipher = PKCS1_v1_5.new(rsa_key)
    return str(cipher.decrypt(convert_data, "DecryptError"), encoding='utf-8')


def encrypt(data: str, public_key: str) -> str:
    rsa_key = RSA.importKey(base64.b64decode(public_key))
    cipher = PKCS1_v1_5.new(rsa_key)
    result_data = cipher.encrypt(bytes(data, encoding="utf8"))
    return str(base64.b64encode(result_data), encoding='utf-8')


if __name__ == '__main__':
    encrypt_result = encrypt('123456', PUBLIC_KEY)
    print(encrypt_result)
    # encrypt_result = 'RZRbgTEEN9hXVdRab3wbCycIwWqBcGjRk9V2V0nDhKZwjgQ4pWHOu6A=='
    decrypt_data = decrypt(encrypt_result, PRIVATE_KEY)
    print(decrypt_data)
```

### Java代码片段

```java

```

### RSA的秘钥类型汇总

|     PKCS     | 版本 |                             名称                             | 简介                                                         |
| :----------: | ---- | :----------------------------------------------------------: | ------------------------------------------------------------ |
| PKCS #1 [^1] | 2.1  |         RSA密码编译标准（RSA Cryptography Standard）         | 定义了RSA的数理基础、公/私钥格式，以及加/解密、签/验章的流程。1.5版本曾经遭到攻击。 |
|   PKCS #2    | -    |                             撤销                             | 原本是用以规范RSA加密摘要的转换方式，现已被纳入PKCS#1之中。  |
|   PKCS #3    | 1.4  |   DH密钥协议标准（Diffie-Hellman key agreement Standard）    | 规范以DH密钥协议为基础的密钥协议标准。其功能，可以让两方通过金议协议，拟定一把会议密钥(Session key)。 |
|   PKCS #4    | -    |                             撤销                             | 原本用以规范转换RSA密钥的流程。已被纳入PKCS#1之中。          |
|   PKCS #5    | 2.0  |    密码基植加密标准（Password-based Encryption Standard）    | 参见RFC 2898与PBKDF2。                                       |
|   PKCS #6    | 1.5  |   证书扩展语法标准（Extended-Certificate Syntax Standard）   | 将原本X.509的证书格式标准加以扩充。                          |
|   PKCS #7    | 1.5  |  密码消息语法标准（Cryptographic Message Syntax Standard）   | 参见RFC 2315。规范了以公开密钥基础设施（PKI）所产生之签名/密文之格式。其目的一样是为了拓展数字证书的应用。其中，包含了S/MIME与CMS。 |
| PKCS #8 [^2] | 1.2  | 私钥消息表示标准（Private-Key Information Syntax Standard）. | Apache读取证书私钥的标准。                                   |
|   PKCS #9    | 2.0  |           选择属性格式（Selected Attribute Types）           | 定义PKCS#6、7、8、10的选择属性格式。                         |
|   PKCS #10   | 1.7  | 证书申请标准（Certification Request Standard） 参见RFC 2986。 | 规范了向证书中心申请证书之CSR（certificate signing request）的格式。 |
|   PKCS #11   | 2.20 | 密码设备标准接口（Cryptographic Token Interface (Cryptoki)） | 定义了密码设备的应用程序接口（API）之规格。                  |
|   PKCS #12   | 1.0  | 个人消息交换标准（Personal Information Exchange Syntax Standard） | 定义了包含私钥与公钥证书（public key certificate）的文件格式。私钥采密码(password)保护。常见的PFX就履行了PKCS#12。 |
|   PKCS #13   | –    |  椭圆曲线密码学标准（Elliptic curve cryptography Standard）  | 制定中。规范以椭圆曲线密码学为基础所发展之密码技术应用。椭圆曲线密码学是新的密码学技术，其强度与效率皆比现行以指数运算为基础之密码学算法来的优秀。然而，该算法的应用尚不普及。 |
|   PKCS #14   | –    |    拟随机数产生器标准（Pseudo-random Number Generation）     | 制定中。规范拟随机数产生器的使用与设计。                     |
|   PKCS #15   | 1.1  | 密码设备消息格式标准（Cryptographic Token Information Format Standard） | 定义了密码                                                   |

### RSA的格式转换

```bash
# 生成PKCS1私钥
openssl genrsa -out private.pem 1024

# PKCS1私钥转换为PKCS8
openssl pkcs8 -topk8 -inform PEM -in private.pem -outform pem -nocrypt -out pkcs8.pem

# PKCS8私钥转换为PKCS1
openssl rsa -in pkcs8.pem -out pkcs1.pem

# 从PKCS1私钥中生成PKCS8公钥
openssl rsa -in private.pem -pubout -out public.pem

# 从PKCS8私钥中生成PKCS8公钥
openssl rsa -in pkcs8.pem -pubout -out public_pkcs8.pem

# 从PKCS8公钥转换PKCS1公钥
openssl rsa -pubin -in public_pkcs8.pem -RSAPublicKey_out

# 从PKCS1公钥转换PKCS8公钥
openssl rsa -RSAPublicKey_in -in pub_pkcs1.pem -pubout
```

 

 

- 参考链接：[https://daniao.ws/rsa-java-to-python.html](https://daniao.ws/rsa-java-to-python.html)