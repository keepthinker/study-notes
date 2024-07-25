# jasypt
## 解密
java -cp lib/jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringDecryptionCLI  password=fdfdfd  algorithm=PBEWithMD5AndDES input=9bgMMaeq9Cj0XrcQW7MHEA==


## 加密
java -cp lib/jasypt-1.9.2.jar org.jasypt.intf.cli.JasyptPBEStringEncryptionCLI  password=fdfdfd  algorithm=PBEWithMD5AndDES input=fund
