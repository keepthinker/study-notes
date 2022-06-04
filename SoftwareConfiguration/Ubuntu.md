# Ubuntu

## proxy

```shell
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891
```

## github

```shell
 git config --global http.proxy localhost:7890
```

## CA证书生成

### 自签名证书的制作

```shell
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private-nginx-selfsigned.key -out certs-nginx-selfsigned.crt
```

### nginx.conf配置

```shell
server {
  # 最新配置
  listen 443 ssl;
  # listen 443;
  # ssl on;
  ssl_certificate /etc/ssl/ssl-bundle.crt;
  ssl_certificate_key /path/to/your_private.key;
  root /path/to/webroot;
  server_name your_domain.com;

  access_log /var/log/nginx/nginx.vhost.access.log;
  error_log /var/log/nginx/nginx.vhost.error.log;
  location / {
    root /var/www/;
    root  /home/www/public_html/your.domain.com/public/;
    index index.html;
  }
}
```

```shell
# 验证443端口和域名是否在nginx配置成功，先跳过客户端对服务器的证书校验，使用-k或者--insecure
curl -k -H 'Host:www.springboot-example.keepthinker.com'  https://106.14.192.20
```

### 参考

[自签名 SSL 证书](https://www.xtplayer.cn/ssl/self-signed-ssl/#%E8%87%AA%E7%AD%BE%E5%90%8D%E7%B1%BB%E5%9E%8B)

[How To Create a Self-Signed SSL Certificate for Nginx in Ubuntu 16.04 | DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04)

[HTTPS证书生成原理和部署细节 | 小胡子哥的个人网站](https://www.barretlee.com/blog/2015/10/05/how-to-build-a-https-server/)

[How to Install SSL Certificate on NGINX Server](https://phoenixnap.com/kb/install-ssl-certificate-nginx)
