# 安全

## CA证书

### CA证书验证原理

![img](https-certificate.jpg)

### CA 签发证书的过程

- 首先 CA 会把持有者的**公钥、用途、颁发者、有效时间等信息**打成一个包，然后对这些信息进行 Hash 计算，得到一个 Hash 值；
- 然后 CA 会使用自己的私钥将该 Hash 值加密，生成 Certificate Signature，也就是 CA 对证书做了签名；
- 最后将 Certificate Signature 添加在文件证书上，形成数字证书；

### 客户端校验服务端的数字证书的过程

- 首先客户端会使用同样的 Hash 算法获取该证书的 Hash 值 H1；
- 通常浏览器和操作系统中集成了 CA 的公钥信息，浏览器收到证书后可以使用 CA 的公钥解密 Certificate Signature 内容，得到一个 Hash 值 H2 ；
- 最后比较 H1 和 H2，如果值相同，则为可信赖的证书，否则则认为证书不可信。

### 证书信用链

![!certificate-trust-chain](certificate-trust-chain.jpg)

![certificate-verify-chain](certificate-verifyt-chain.jpg)

### 制作证书

#### 自签名证书

```bash
# 1. 生成服务器私钥

openssl genrsa -des3 -out server.key 4096

# 2. 生成证书签名请求

openssl req -new -key server.key -out server.csr

保证Common name跟你的域名或者IP相同

# 3. 对证书签名请求进行签名

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# 4. 生成无需密码的服务器私钥

openssl rsa -in server.key -out server.key.insecure

# 5.重命名私钥文件. 其中server.key.insecure会被比如nginx使用

mv server.key server.key.secure

mv server.key.insecure server.key


# 简单的方法一步生成私钥及自签名证书

openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout server.key -out server.crt
```

#### 私有CA证书

```bash
# 1. 创建CA私钥

openssl genrsa -des3 -out ca.key 4096

# 2. 可选步骤，删除私钥中的密码：openssl rsa -in ca.key -out ca.key

# 3. 生成CA的自签名证书

openssl req -new -x509 -days 365 -key ca.key -out ca.crt

# 其实CA证书就是一个自签名证书

# 4. 生成服务端私钥，可以参考"自签名证书"中的第4步骤，生成一个无密码的私钥

openssl genrsa -des3 -out server.key 4096

# 5. 需要签名的证书(服务端)生成证书签名请求，可以增加参数，-subj "/C=CN/ST=Shanghai/L=Shanghai/O=keepthiner/OU=spring/CN=*.keepthinker.com"，这样无需后续一个个输入，-config增加一些其他参数

# 注意chrome等浏览器验证证书合法需要subjectAltName，故采用如下的方式生成csr文件。根据提示，如缺少文件或目录则添加，比如添加index.txt空内容文件，demoCA/serial里存放偶数序列号，具体要求的文件位置，可以在openssl.conf的[ CA_default ]下方进行修改
openssl req -new \
    -sha256 \
    -key server-keepthinker-springboot-example.key \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=keepthinker/OU=spring/CN=*.keepthinker.com" \
    -reqexts SAN \
    -config <(cat /etc/ssl/openssl.cnf \
        <(printf "[SAN]\nsubjectAltName=DNS:*.keepthinker.com,DNS:*.springboot-example.keepthinker.com,DNS:106.14.192.20")) \
    -out server-keepthinker-springboot-example.csr

## 使用cnf文件
openssl req -new \
    -sha256 \
    -key server-keepthinker-springboot-example.key \
    -subj "/C=CN/ST=Shanghai/L=Shanghai/O=keepthinker/OU=spring/CN=*.keepthinker.com" \
    -reqexts SAN \
    -config openssl-keepthinker-springboot-example.cnf \
    -out server-keepthinker-springboot-example.csr

# 一般生成证书csr的方式
openssl req -new -key server.key -out server.csr

# 证书签名请求中的Common Name必须区别与CA的证书里面的Common Name

# 6. 用CA证书给步骤四中的签名请求进行签名

# 注意chrome等浏览器验证证书合法需要subjectAltName，故采用如下的方式生成crt文件
openssl ca -in server-keepthinker-springboot-example.csr \
    -md sha256 \
    -keyfile  ca-keepthinker-with-password.key \
    -cert ca-keepthinker.crt \
    -extensions SAN \
    -config <(cat /etc/ssl/openssl.cnf \
        <(printf "[SAN]\nsubjectAltName=DNS:*.keepthinker.com,DNS:*.springboot-example.keepthinker.com,DNS:106.14.192.20")) \
    -out server-keepthinker-springboot-example.crt

## 使用cnf文件
openssl ca -in server-keepthinker-springboot-example.csr \
    -md sha256 \
    -extensions SAN \
    -config openssl-keepthinker-springboot-example.cnf \
    -keyfile  ca-keepthinker-with-password.key \
    -cert ca-keepthinker.crt \
    -out server-keepthinker-springboot-example.crt
## 如果提示ERROR:There is already a certificate，如果需要再次生成证书，可以直接>demoCA/index.txt来清空文件，使可以再次生成证书。

# 一般生成证书crt的方式
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
```

### 查看证书等信息

```bash
# 查看私钥信息

openssl rsa -noout -text -in server.key

# 查看签名请求信息

openssl req -noout -text -in server.csr

# 查看证书信息

openssl x509 -noout -text -in server.crt

# 将crt文件转换为pem

openssl x509 -in server.crt -out server.pem -outform PEM

# 验证一个证书是否是某一个CA签发

openssl verify -CAfile ca.crt server.crt

# 模拟一个ssl客户端访问ssl服务器

openssl s_client -connect 192.168.0.1:443 -cert client.crt -key client.key

# 特定证书(--cacert指定的证书)验证目标https地址证书通过，则可以正常访问
curl --cacert ca-keepthinker.pem https://www.springboot-example.keepthinker.com

# 转换crt成pem
openssl x509 -in ca.crt -out ca.pem
```

### Windows证书导入

```bash
>> certmgr
```

### SAN(Subject Alternative Name)

Subject Alternative Name (SAN) is an extension to X.509 that lets you specify additional host names (values) to be protected by a single SSL certificate using a subjectAltName field. It allows more than one host to use the same copy of a single certificate. At the server-level, you can create multiple virtual hosts and add these hosts to the subjectAltName field of the certificate. You generate a certificate with SAN and the clients can connect to the server using subjectAltName. Whenever HTTPS request comes to any of the virtual host, the server uses the same certificate for SSL handshake.

```bash
# Before you generate the digital certificate using the pkiutil utility, open the pscpki.cnf file in the %DLC%\keys\policy location and add the subjectAltName values as follows under the x509v3_extensions section:
  subjectAltName = @alt_names

# Add the domain names in the alt_names section as follows:
 [alt_names]
          DNS.1 = <value>
          DNS.2 = <value>

# 如果是ubuntu，openssl req如果参数添加-reqexts SAN，那么打开/etc/ssl/openssl.conf文件(也可以指定用-config文件位置)，添加上述alt_names等配置，格式和上面一致。 并且需要在subjectAltName前补充[ SAN ]。具体命令，参考上文。
```

### 问题分析

#### 阿里云

如果域名没有备案，发起https中SSL的Client Hello请求会被阿里云直接拒绝直接返回RST

### 参考

https://www.zhihu.com/question/37370216/answer/1914075935

[自签名证书和私有CA证书的制作](https://blog.csdn.net/shanghongshen/article/details/120923968)

[OpenSSL创建带SAN扩展的证书并进行CA自签](https://www.jianshu.com/p/7ade7317bc6e)

[怎么让 chrome 信任自签名证书（亲测有效）](https://blog.csdn.net/freeabc/article/details/109737552)

[Adding Subject Alternative Name (SAN) to a digital certificate](https://docs.progress.com/zh-CN/bundle/openedge-security-auditing-introduction-117/page/Adding-Subject-Alternative-Name-SAN-to-a-digital-certificate.html)

[How to create a certificate using OpenSSL with Subject Alternative Name field (SAN)](https://help.bizagi.com/bpm-suite/en/index.html?subjectaltname_support.htm)

[openssl官网](https://www.openssl.org/)

[Provide subjectAltName to openssl directly on the command line](https://security.stackexchange.com/questions/74345/provide-subjectaltname-to-openssl-directly-on-the-command-line)

## JWT(JSON Web Token)

1. 单体应用建议用HMAC方式对JWT进行加密签名。

2. 在多方系统或者授权服务与资源服务分离的分布式应用中，建议用公钥加密签名的方式。

## 参考

[凭证 | 凤凰架构](https://icyfenix.cn/architect-perspective/general-architecture/system-security/credentials.html#jwt)