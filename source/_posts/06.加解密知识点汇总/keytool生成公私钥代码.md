# keytool生成公私钥指令

```sh
keytool -genkeypair -keysize 1024 -validity 3650 -alias "privateKey" -keystore "privateKeys.keystore" -storepass "pubwd123456" -keypass "priwd123456" -dname "CN=localhost, OU=localhost, O=localhost, L=SH, ST=SH, C=CN"
```

```sh
 keytool -exportcert -alias "privateKey" -keystore "privateKeys.keystore" -storepass "pubwd123456" -file "certfile.cer"
```

```sh
 keytool -import -alias "publicCert" -file "certfile.cer" -keystore "publicCerts.keystore" -storepass "pubwd123456"
```

