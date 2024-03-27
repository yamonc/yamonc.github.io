---
title: JDK的keytool工具总结
date: 2023-07-12 20:25:59
tags: [keytool, 加密工具类]
categories: [加密算法]
recommend: true
locate: [天津]
cover: https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307122022899.jpeg
comment: 是
keywords: 
---
# JDK的keytool工具总结

![a very tall mountain with a very steep face](https://markdown-image-bed.oss-cn-beijing.aliyuncs.com/202307122022899.jpeg)

## keytool命令说明

| 项目                                | 详细                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| -certreq                            | 创建证书请求                                                 |
| -changealias                        | 变更证书私钥                                                 |
| -exportcert                         | 导出证书                                                     |
| -genkeypair                         | 创建密钥对                                                   |
| -genseckey                          | 创建加密密钥                                                 |
| -gencert                            | 依据证书请求创建证书                                         |
| -importcert                         | 导入证书或者证书链                                           |
| -importpass                         | 导入密码                                                     |
| -importkeystore                     | 从其他keystore中导入一个或者所有条目                         |
| -printcert                          | 打印证书内容                                                 |
| -printcertreq                       | 打印证书请求的内容                                           |
| -printcrl                           | 打印CRL文件内容                                              |
| -genkey                             | 在用户主目录中创建一个默认文件".keystore",还会产生一个mykey的别名，mykey中包含用户的公钥、私钥和证书 |
| -alias                              | 产生别名 缺省值"mykey"                                       |
| -keystore                           | 指定密钥库的名称(产生的各类信息将不在.keystore文件中)        |
| -keyalg                             | 指定密钥的算法 (如 RSA DSA（如果不指定默认采用DSA）)         |
| -validity                           | 指定创建的证书有效期多少天 缺省值90天                        |
| -keysize                            | 指定密钥长度 缺省值1024                                      |
| -storepass                          | 指定密钥库的密码(获取keystore信息所需的密码)                 |
| -keypass                            | 指定别名条目的密码(私钥的密码)                               |
| -dname                              | 指定证书拥有者信息 例如： “CN=名字与姓氏,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码” |
| -list                               | 显示密钥库中的证书信息 keytool -list -v -keystore 指定keystore -storepass 密码 |
| -v                                  | 显示密钥库中的证书详细信息                                   |
| -export                             | 将别名指定的证书导出到文件 keytool -export -alias 需要导出的别名 -keystore 指定keystore -file 指定导出的证书位置及证书名称 -storepass 密码 |
| -file	参数指定导出到文件的文件名 | 参数指定导出到文件的文件名                                   |
| -delete                             | 删除密钥库中某条目 keytool -delete -alias 指定需删除的别 -keystore 指定keystore -storepass 密码 |
| -printcert                          | 查看导出的证书信息 keytool -printcert -file 证书名称         |
| -keypasswd                          | 修改密钥库中指定条目口令                                     |
| -storepasswd                        | 修改keystore口令 keytool -storepasswd -keystore keystore的FULLPATH -storepass 原始密码  -new 新密码 |
| -import                             | 将已签名数字证书导入密钥库 keytool -import -alias 指定导入条目的别名 -keystore 指定keystore -file 需导入的证书 |

## 证书管理

证书的发行有专门的CA机构，但是基本上都是要付费的。一般来说除非是非常正式的项目，一般的项目很多情况下使用自发行的证书即可。

| 认证方式 | 证书种类              | 详细                                                         |
| -------- | --------------------- | ------------------------------------------------------------ |
| 单向认证 | 服务器端证书          | 客户端对服务器端的证书进行认证                               |
| 双向认证 | 服务端证书/客户端证书 | 客户端对服务器端的证书进行认证,同时服务器端对客户端的证书也进行认证 |

## keystore生成

| 项目         | 详细                 |
| ------------ | -------------------- |
| alias名称    | kstore               |
| keypass      | init123              |
| 算法         | RSA                  |
| 秘钥长度     | 2048                 |
| 有效期限(天) | 30                   |
| 保存路径     | /tmp/kstore.keystore |
| storepass    | init234              |

执行相应的指令：

```bash
keytool -genkey -alias kstore -keypass init123 -keyalg RSA -keysize 2048 -validity 30 -keystore /tmp/kstore.keystore -storepass init234
```

输入一些必要的信息：

```bash
[root@liumiaocn ~]# keytool -genkey -alias kstore -keypass init123 -keyalg RSA -keysize 2048 -validity 30 -keystore /tmp/kstore.keystore -storepass init234
What is your first and last name?
  [Unknown]:  michael
What is the name of your organizational unit?
  [Unknown]:  liumiaocn
What is the name of your organization?
  [Unknown]:  ngo
What is the name of your City or Locality?
  [Unknown]:  dalian
What is the name of your State or Province?
  [Unknown]:  liaoning
What is the two-letter country code for this unit?
  [Unknown]:  CN
Is CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN correct?
  [no]:  yes

[root@liumiaocn ~]#
```

## keystore确认

可以使用list子命令对keystore确认详细信息。

```bash
keytool -list  -v -keystore /tmp/kstore.keystore -storepass init234
```

执行后参照：

```sh
[root@liumiaocn ~]# keytool -list  -v -keystore /tmp/kstore.keystore -storepass init234

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 1 entry

Alias name: kstore
Creation date: Mar 10, 2017
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Issuer: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Serial number: 58700a1
Valid from: Fri Mar 10 17:46:00 EST 2017 until: Sun Apr 09 18:46:00 EDT 2017
Certificate fingerprints:
         MD5:  C9:88:B5:3E:62:F1:31:4D:8B:81:9C:45:90:F1:0F:CF
         SHA1: 59:C9:D3:3F:07:80:73:7C:7E:43:94:3B:E5:43:61:FF:14:F1:1A:CC
         SHA256: 32:71:6C:1E:1F:F6:23:01:66:81:92:36:C8:6F:E3:8D:5B:32:C4:F2:10:94:D0:3D:8C:07:5B:91:7A:59:B2:56
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 06 91 44 93 93 46 D0 EE   A9 B3 9C A6 6C 1A BD D4  ..D..F......l...
0010: E3 EA 74 74                                        ..tt
]
]



*******************************************
*******************************************


[root@liumiaocn ~]#
```

## 证书导出

使用以下命令导出证书：

```sh
keytool -export -alias kstore -keystore /tmp/kstore.keystore -file /tmp/kstore.crt -rfc -storepass init234
```

注意: storepass的密码是/tmp/kstore.keystore生成时创建的密码，此处是作确认用，输入错误会提示：Keystore was tampered with, or password was incorrect

```sh
[root@liumiaocn ~]# keytool -export -alias kstore -keystore /tmp/kstore.keystore -file /tmp/kstore.crt -rfc -storepass init234
Certificate stored in file </tmp/kstore.crt>
[root@liumiaocn ~]# file /tmp/kstore.crt
/tmp/kstore.crt: PEM certificate
[root@liumiaocn ~]#
[root@liumiaocn ~]#
[root@liumiaocn ~]# cat /tmp/kstore.crt
-----BEGIN CERTIFICATE-----
MIIDaTCCAlGgAwIBAgIEBYcAoTANBgkqhkiG9w0BAQsFADBlMQswCQYDVQQGEwJD
TjERMA8GA1UECBMIbGlhb25pbmcxDzANBgNVBAcTBmRhbGlhbjEMMAoGA1UEChMD
bmdvMRIwEAYDVQQLEwlsaXVtaWFvY24xEDAOBgNVBAMTB21pY2hhZWwwHhcNMTcw
MzEwMjI0NjAwWhcNMTcwNDA5MjI0NjAwWjBlMQswCQYDVQQGEwJDTjERMA8GA1UE
CBMIbGlhb25pbmcxDzANBgNVBAcTBmRhbGlhbjEMMAoGA1UEChMDbmdvMRIwEAYD
VQQLEwlsaXVtaWFvY24xEDAOBgNVBAMTB21pY2hhZWwwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQDL/LZP2FH8jybQD7KvKaqo0TS17xr8isWIkYcjVkvP
T8ZoijWxSLckLI8r83g1Az+obfBuqDlbP/C2qGqf64MGw4mh6/CXnYNgvTOzgLOq
xFgCSQebBhGx0u6gDCpgkoOdTzljDs6YR7Jv3rhIUodsPxc+Bzf8NXbDi1i6rvi5
UW7ijLYFH1jAozTOlLtVWK1cw97BlXXcYhUEBsO682UzYvOeqCuPGcXKX7v1CsND
j+OWf+kfAZ1RF7e3/8IC/FBuAVKHbMD9jWknwAMI7NqGx4Ej6K0LFL6C4XmZMHla
sznT7Eyri0YS5jdWYI+duOEmcbh7MR1FzsZzcw8mmSKPAgMBAAGjITAfMB0GA1Ud
DgQWBBQGkUSTk0bQ7qmznKZsGr3U4+p0dDANBgkqhkiG9w0BAQsFAAOCAQEAdRpS
qAtxZ5JEzN/upzlT6Cp2kK6k7ZQ8ezKYDdUgBtqvMC/TaHwJCMHnvr0aSs24o8SI
v0vmX01RXf5qzMmelMiJCA2EyRFIsKwKE6fWvyRaK1L5NiKbbivqiOHlvcw5pZfK
iC/Cy6W0Y7KY4AtfNreAX7lmxQ611sc6F/Dz5rFv8s5gSaBg6oxc3HnXWYkR9iGu
lYJUhV4tanXsgyhS1/N6PiLbwZ+8ryactI9XgCmao+WQ1M0uhEITZFr8TuRkrG9K
fmvZUD+STRGDh4Us447vezeCw3s/r2rNsvxSpPhRd9PlPJShLe8lVM+bpkdilGwk
yOAewyPCWGtaaWXwIQ==
-----END CERTIFICATE-----
[root@liumiaocn ~]#
```

## 证书确认

生成的证书的格式是PEM certificate：

```sh
keytool -printcert -file /tmp/kstore.crt
```

执行参照：

```sh
[root@liumiaocn ~]# keytool -printcert -file /tmp/kstore.crt
Owner: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Issuer: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Serial number: 58700a1
Valid from: Fri Mar 10 17:46:00 EST 2017 until: Sun Apr 09 18:46:00 EDT 2017
Certificate fingerprints:
         MD5:  C9:88:B5:3E:62:F1:31:4D:8B:81:9C:45:90:F1:0F:CF
         SHA1: 59:C9:D3:3F:07:80:73:7C:7E:43:94:3B:E5:43:61:FF:14:F1:1A:CC
         SHA256: 32:71:6C:1E:1F:F6:23:01:66:81:92:36:C8:6F:E3:8D:5B:32:C4:F2:10:94:D0:3D:8C:07:5B:91:7A:59:B2:56
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 06 91 44 93 93 46 D0 EE   A9 B3 9C A6 6C 1A BD D4  ..D..F......l...
0010: E3 EA 74 74                                        ..tt
]
]

[root@liumiaocn ~]#
```

## 证书导入

使用指令将证书导入到keystore中：

```sh
keytool -import -alias aliascrt -file /tmp/kstore.crt -keystore /tmp/kstore.keystore -storepass init234 -keypass init123
```

导入前的确认：

```sh
[root@liumiaocn tmp]# keytool -list  -v -keystore /tmp/kstore.keystore -storepass init234

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 1 entry

Alias name: kstore
Creation date: Mar 10, 2017
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Issuer: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Serial number: 58700a1
Valid from: Fri Mar 10 17:46:00 EST 2017 until: Sun Apr 09 18:46:00 EDT 2017
Certificate fingerprints:
         MD5:  C9:88:B5:3E:62:F1:31:4D:8B:81:9C:45:90:F1:0F:CF
         SHA1: 59:C9:D3:3F:07:80:73:7C:7E:43:94:3B:E5:43:61:FF:14:F1:1A:CC
         SHA256: 32:71:6C:1E:1F:F6:23:01:66:81:92:36:C8:6F:E3:8D:5B:32:C4:F2:10:94:D0:3D:8C:07:5B:91:7A:59:B2:56
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 06 91 44 93 93 46 D0 EE   A9 B3 9C A6 6C 1A BD D4  ..D..F......l...
0010: E3 EA 74 74                                        ..tt
]
]



*******************************************
*******************************************


[root@liumiaocn tmp]#
```

导入之后的确认：

```sh
[root@liumiaocn tmp]# keytool -list  -v -keystore /tmp/kstore.keystore -storepass init234

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 2 entries

Alias name: kstore
Creation date: Mar 10, 2017
Entry type: PrivateKeyEntry
Certificate chain length: 1
Certificate[1]:
Owner: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Issuer: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Serial number: 58700a1
Valid from: Fri Mar 10 17:46:00 EST 2017 until: Sun Apr 09 18:46:00 EDT 2017
Certificate fingerprints:
         MD5:  C9:88:B5:3E:62:F1:31:4D:8B:81:9C:45:90:F1:0F:CF
         SHA1: 59:C9:D3:3F:07:80:73:7C:7E:43:94:3B:E5:43:61:FF:14:F1:1A:CC
         SHA256: 32:71:6C:1E:1F:F6:23:01:66:81:92:36:C8:6F:E3:8D:5B:32:C4:F2:10:94:D0:3D:8C:07:5B:91:7A:59:B2:56
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

\#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 06 91 44 93 93 46 D0 EE   A9 B3 9C A6 6C 1A BD D4  ..D..F......l...
0010: E3 EA 74 74                                        ..tt
]
]



*******************************************
*******************************************


Alias name: aliascrt
Creation date: Mar 10, 2017
Entry type: trustedCertEntry

Owner: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Issuer: CN=michael, OU=liumiaocn, O=ngo, L=dalian, ST=liaoning, C=CN
Serial number: 58700a1
Valid from: Fri Mar 10 17:46:00 EST 2017 until: Sun Apr 09 18:46:00 EDT 2017
Certificate fingerprints:
         MD5:  C9:88:B5:3E:62:F1:31:4D:8B:81:9C:45:90:F1:0F:CF
         SHA1: 59:C9:D3:3F:07:80:73:7C:7E:43:94:3B:E5:43:61:FF:14:F1:1A:CC
         SHA256: 32:71:6C:1E:1F:F6:23:01:66:81:92:36:C8:6F:E3:8D:5B:32:C4:F2:10:94:D0:3D:8C:07:5B:91:7A:59:B2:56
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

\#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 06 91 44 93 93 46 D0 EE   A9 B3 9C A6 6C 1A BD D4  ..D..F......l...
0010: E3 EA 74 74                                        ..tt
]
]



*******************************************
*******************************************
```

## 证书删除

```sh
keytool -delete -alias aliascrt -keystore /tmp/kstore.keystore -storepass init234
```







## 参考

[Java Keytool工具简介](https://blog.csdn.net/liumiaocn/article/details/61921014)
