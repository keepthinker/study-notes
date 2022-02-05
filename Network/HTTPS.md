# HTTPS

**超文本传输安全协议**（英语：**H**yper**T**ext **T**ransfer **P**rotocol **S**ecure，缩写：**HTTPS**；常称为HTTP over TLS、HTTP over SSL或HTTP Secure）是一种通过[计算机网络](https://zh.wikipedia.org/wiki/計算機網絡)进行安全通信的[传输协议](https://zh.wikipedia.org/wiki/網路傳輸協定)。HTTPS经由[HTTP](https://zh.wikipedia.org/wiki/HTTP)进行通信，但利用[SSL/TLS](https://zh.wikipedia.org/wiki/传输层安全)来[加密](https://zh.wikipedia.org/wiki/加密)数据包。



### Network layers

HTTP operates at the highest layer of the [TCP/IP model](https://en.wikipedia.org/wiki/TCP/IP_model)—the [application layer](https://en.wikipedia.org/wiki/Application_layer); as does the [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) security protocol (operating as a lower sublayer of the same layer), which encrypts an HTTP message prior to transmission and decrypts a message upon arrival. Strictly speaking, HTTPS is not a separate protocol, but refers to the use of ordinary [HTTP](https://en.wikipedia.org/wiki/HTTP) over an [encrypted](https://en.wikipedia.org/wiki/Encryption) SSL/TLS connection.

HTTPS encrypts all message contents, including the HTTP headers and the request/response data. With the exception of the possible [CCA](https://en.wikipedia.org/wiki/Chosen-ciphertext_attack) cryptographic attack described in the [limitations](https://en.wikipedia.org/wiki/HTTPS#Limitations) section below, an attacker should at most be able to discover that a connection is taking place between two parties, along with their domain names and IP addresses.



## HandShake procedure

1. Client **sends a "ClientHello" message** that lists information such as the **SSL/TLS version**, **crytographic algorithms**, **data compression algorithm methods** supported by the client.
2. Server responds with a "ServerHello" message that contains the **cryptographic algorithms** chosen by server from the list provided by the client, **the session ID**, **Server's digital certificate** and **public key**.
3. Client will contack server's CA and verify the server's digital certificate, thus comfirming the authenticity of the web server.
4. Client **send a shared secret key** which is encryted with the server's public key.
5. Client **sends a "finished" message** which is encrypted with the secret key indicating that the client part of handshake is complete.
6. Server **response to client with a "finished" message** which is encrypted with the secret key indicating that the server part of handshake is complete.

Once the handshake is done,  the server and client can now exchange messages that are symmetrically encrypted with the shared secret key.



# An overview of the SSL or TLS handshake

The SSL or TLS handshake enables the SSL or TLS client and server to establish the secret keys with which they communicate.

This section provides a summary of the steps that enable the SSL or TLS client and server to communicate with each other:

- Agree on the version of the protocol to use.
- Select cryptographic algorithms.
- Authenticate each other by exchanging and validating digital certificates.
- Use asymmetric encryption techniques to generate a shared secret key, which avoids the key distribution problem. SSL or TLS then uses the shared key for the symmetric encryption of messages, which is faster than asymmetric encryption.

For more information about cryptographic algorithms and digital certificates, refer to the related information.

This section does not attempt to provide full details of the messages exchanged during the SSL handshake. In overview, the steps involved in the SSL handshake are as follows:

1. The SSL or TLS client **sends a “client hello” message** that lists cryptographic information such as **the SSL or TLS version and, in the client's order of preference, the CipherSuites supported by the client**. The message also contains a random byte string that is used in subsequent computations. The protocol allows for the “client hello” to **include the data compression methods** supported by the client.
2. The SSL or TLS server responds with a **“server hello” message** that contains the **[CipherSuite](https://en.wikipedia.org/wiki/Cipher_suite) chosen by the server from the list provided by the client, the session ID**, and another random byte string. **The server also sends its digital certificate(Contains Public key, RSA public key forexample).** **If the server requires a digital certificate for client authentication, the server sends a “client certificate request”** that includes a list of the types of certificates supported and the Distinguished Names of acceptable Certification Authorities (CAs).
3. The SSL or TLS **client verifies the server's digital certificate**. For more information, see [How SSL and TLS provide identification, authentication, confidentiality, and integrity](https://www.ibm.com/docs/en/SSFKSJ_7.5.0/com.ibm.mq.sec.doc/q009940_.html).
4. The SSL or TLS client **sends the random byte string that enables both the client and the server to compute the (AES for example)secret key to be used for encrypting subsequent message data.** The random byte string itself is **encrypted with the server's public key.**
5. **If the SSL or TLS server sent a “client certificate request”, the client sends a random byte string encrypted with the client's private key, together with the client's digital certificate**,** or a “no digital certificate alert”. This alert is only a warning, but with some implementations the handshake fails if client authentication is mandatory.
6. **The SSL or TLS server verifies the client's certificate.** For more information, see [How SSL and TLS provide identification, authentication, confidentiality, and integrity](https://www.ibm.com/docs/en/SSFKSJ_7.5.0/com.ibm.mq.sec.doc/q009940_.html).
7. The SSL or TLS **client sends the server a “finished” message**, which is **encrypted with the secret key**, indicating that the client part of the handshake is complete.
8. The SSL or TLS **server sends the client a “finished” message**, which is **encrypted with the secret key**, indicating that the server part of the handshake is complete.
9. For the duration of the SSL or TLS session, the server and client can now exchange messages that are symmetrically encrypted with the shared secret key.
10. 

[Figure 1](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=ssl-overview-tls-handshake#q009930___q009930_6) illustrates the SSL or TLS handshake.

Figure 1. Overview of the SSL or TLS handshake

![img](https-handshake.gif)



[An overview of the SSL or TLS handshake](https://www.ibm.com/docs/en/ibm-mq/7.5?topic=ssl-overview-tls-handshake)

[How SSL works?](https://www.tutorialsteacher.com/https/how-ssl-works)

[SSL Handshake explained](https://medium.com/@kasunpdh/ssl-handshake-explained-4dabb87cdce)



## 参考文献

[超文本传输安全协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)