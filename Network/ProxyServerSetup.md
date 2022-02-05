Shadowsocks(ss是什么)安装与卸载
BY CDD113 · DECEMBER 22, 2016

前两天，搭建的ss（ss就是shadowsocks）突然翻不了墙了。忙，太忙，忙着约会，一直没有管它。今天想把搞好，突然发现，不知道怎么搞好。想想还是卸载了，重新安装吧。

Shadowsocks卸载命令很简单：

pip uninstall shadowsocks
安装网上也很多：

centos 安装：
```
yum install epel-release
yum update
yum install m2crypto python-setuptools
easy_install pip (如果此处报错，改用yum install python-pip)
```

pip install shadowsocks
如果提示要升级pip，最好不要升级。请忽略以下升级命令。

pip install --upgrade pip
继续执行命令

vi /etc/shadowsocks.json
输入以下内容：yourpassword替换成shadowsocks的密码。0.0.0.0可替换成自己的ip
```
{
    "server":"0.0.0.0",
    "server_port":18080,
    "local_port":1080,
    "password":"speant",
    "timeout":600,
    "method":"rc4-md5"
}
```
继续执行命令

vi /etc/supervisord.conf
将以下内容，粘贴到末尾

[program:shadowsocks]
command=ssserver -c /etc/shadowsocks.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/shadowsocks.log
继续执行命令

vi /etc/rc.local
将以下内容粘贴进去，不要放到#号后面，要另起一行

service supervisord start
重启一下vps
reboot命令

停止，启动命令:

ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
要查看ssservice的参数帮助：

ssservice -h
SS 客户端下载。里面有三个版本，windows和android的。如果你是win xp就用低版本的，没有ios的。下面是百度盘的下载地址

链接: https://pan.baidu.com/s/1bKNXKE 密码: q4yw

（如果链接不可用，请q我：121136374）

ss-local -s 161.117.227.123 -p 18080 -l 18080 -k speant -m rc4-md5

## 安装pip
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
