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



[How SSL works?](https://www.tutorialsteacher.com/https/how-ssl-works)

[SSL Handshake explained](https://medium.com/@kasunpdh/ssl-handshake-explained-4dabb87cdce)



## 参考文献

[超文本传输安全协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%AE%89%E5%85%A8%E5%8D%8F%E8%AE%AE)