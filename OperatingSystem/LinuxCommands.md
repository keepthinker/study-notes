# 用户与用户组

## 组操作

### 创建组

groupadd  test

增加一个test组

### 修改组

groupmod -n test2  test

将test组的名子改成test2

### 删除组

groupdel test2

删除 组test2

### 查看组

- 查看当前登录用户所在的组 groups，查看apacheuser所在组groups apacheuser

- 查看所有组 cat /etc/group

- 有的linux系统没有/etc/group文件的，这个时候看下面的这个方法

cat /etc/passwd |awk -F [:] '{print $4}' |sort|uniq | getent group |awk -F [:] '{print $1}'

这里用到一个命令是getent,可以通过组ID来查找组信息,如果这个命令没有的话,那就很难查找,系统中所有的组了.

## 用户操作

### 增加用户

useradd test

passwd test

增加用户test，有一点要注意的，useradd增加一个用户后，不要忘了给他设置密码，不然不能登录的。

### 修改用户

usermod -d /home/test -G test2 test

将test用户的登录目录改成/home/test，并加入test2组，注意这里是大G。

gpasswd -a test test2 将用户test加入到test2组

gpasswd -d test test2 将用户test从test2组中移出

### 删除用户

userdel test

将test用户删除

### 查看用户

#### 查看当前登录用户

who

#### 查看自己的用户名

whoami

#### 查看单个用户信息

id apacheuser

#### 查看用户登录记录

last 查看登录成功的用户记录
lastb 查看登录不成功的用户记录

#### 查看所有用户

cut -d : -f 1 /etc/passwd
cat /etc/passwd |awk -F \: '{print $1}'

# 系统信息查看

## 查看位数命令

uname -a

file /bin/ls

cat /proc/version

## 查看系统版本

lsb_release -a

cat /etc/os-release

cat /etc/redhat-release

## 查看内核版本

cat /proc/version

uname -a

# 修改主机名

相关文件位置: /etc/hostname, /etc/sysconfig/network
同时设置hosts: /etc/hosts增加${ip} ${hostname}

## 临时设置主机名

hostname ${hostname}

echo ${hostname} > /proc/sys/kernel/hostname

sysctl kernel.hostname ${hostname}

## 永久设置

直接设置到/etc/hostname, 并且/etc/sysconfig/network里设置HOSTNAME=${hostname}
。因为不同Linux发行版本不同，所以两处都设置。

# 重启服务

service ${服务命令} restart

## 例子

service sshd restart

# 查找文件

## find

```
// 查找指定目录dir指定深度depth和指定regexp名称的文件
find $dir -maxdepth $depth -name "$regexp"
```

## 设置vip

```
// 设置
ifconfig eth0:1 192.168.0.107 netmask 255.255.0.0

// 删除
ifconfig eth0:1 down
```

# 防火墙

## 开启或禁用

```shell
systemctl start firewalld # 启动

systemctl status firewalld # 或者 firewall-cmd --state 查看状态

systemctl disable firewalld # 停止

systemctl stop firewalld # 禁用

# ufw 为 ubuntu 命令
ufw enable # 开启防火墙

ufw disable # 关闭防火墙

ufw status # 查看防火墙状态

sudo ufw allow 22                  # 开放22端口
sudo ufw delete allow 21           # 关闭21端口
sudo ufw allow 8001/tcp            # 指定开放8001的tcp协议
sudo ufw delete allow 8001/tcp     # 关闭8001的tcp端口
sudo ufw reload                    # 重启ufw防火墙，使配置生效
sudo ufw allow from 192.168.121.1         # 指定ip为192.168.121.1的计算机操作所有端口
sudo ufw delete allow from 192.168.121.1  # 关闭指定ip为192.168.121.1的计算机操作所有端口
sudo ufw allow from 192.168.121.2 to any port 3306  # 开放指定ip为192.168.121.2的计算机访问本机的3306端口
sudo ufw delete allow from 192.168.121.2 to any port 3306   # 关闭指定ip为192.168.121.2的计算机对本机的3306端口的操作
sudo ufw allow from 192.168.1.1/24 to any port 3306 # 开放指定子网

# 打开443/TCP端口
firewall-cmd --add-port=443/tcp

# 永久打开3690/TCP端口
firewall-cmd --permanent --add-port=3690/tcp

# 永久关闭端口
firewall-cmd --remove-port=80/tcp --permanent

# 永久打开端口好像需要reload一下，临时打开好像不用，如果用了reload临时打开的端口就失效了其它服务也可能是这样的，这个没有测试
firewall-cmd --reload

# 查看防火墙，添加的端口也可以看到
firewall-cmd --list-all
```

### 

## 查看进程和端口

```shell
lsof -i:8080 # 查看8080端口占用
lsof abc.txt # 显示开启文件abc.txt的进程
lsof -c abc # 显示abc进程现在打开的文件
lsof -c -p 1234 # 列出进程号为1234的进程所打开的文件
lsof -g gid # 显示归属gid的进程情况
lsof +d /usr/local/ # 显示目录下被进程开启的文件
lsof +D /usr/local/ # 同上，但是会搜索目录下的目录，时间较长
lsof -d 4 # 显示使用fd为4的进程
lsof -i -U # 显示所有打开的端口和UNIX domain文件
lsof -u username # 查看被用户username打开的文件
lsof -n # 不将ip转换成hostname
lsof -P #此参数禁止将port number转换为service name,预设为转换   
lsof -i TCP -n -P  # 禁止转换域名和端口，查看tcp情况

# 查看某进程网络端口例子
lsof -i -n -P | grep mongod
```

# vim命令

vim -b temp.txt 那么此时将可以看到carriage return字符(\r)，显示为^M。进入vim界面后，数据:set list那么可以看到line feed字符(\n)，显示为$，tab键显示为^I

输入carriage return用clt+v, ctl+Enter键。

## 参考文献

[Linux firewall-cmd 命令详解](https://blog.csdn.net/GMingZhou/article/details/78090963)

[Ubuntu系统中防火墙的使用和开放端口_Aaron_Run的博客-CSDN博客_ufw开放端口](https://blog.csdn.net/qq_36938617/article/details/95234909)

## 设置代理

```shell
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891
```

## 下载

```shell
# 设置代理下载时，不校验证书
wget --no-check-certificate https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
```

## 切割文件

```bash
## 指定文件大小来切割，每个文件大小500m，生成的新文件的文件名是newfile后面加上按照aa，ab，ac……来排序的
split -b 500m log.txt newfile

## 切开的文件合起来
cat newfile* > orifile
```

## 字符串修改

```bash
# 替换文件中的所有匹配项
sed -i 's/原字符串/替换字符串/g' filename
```

## 压缩文件命令

```bash
tar -cf all.tar *.jpg # 这条命令是将所有.jpg的文件打成一个名为all.tar的包。-c是表示产生新的包，-f指定包的文件名。
tar -rf all.tar *.gif # 这条命令是将所有.gif的文件增加到all.tar的包里面去。-r是表示增加文件的意思。 
tar -uf all.tar logo.gif # 这条命令是更新原来tar包all.tar中logo.gif文件，-u是表示更新文件的意思。 
tar -tf all.tar # 这条命令是列出all.tar包中所有文件，-t是列出文件的意思 
tar -xf all.tar # 这条命令是解出all.tar包中所有文件，-x是解开的意思


tar –cvf jpg.tar *.jpg # 将目录里所有jpg文件打包成tar.jpg
tar –czf jpg.tar.gz *.jpg # 将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar –cjf jpg.tar.bz2 *.jpg # 将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar –cZf jpg.tar.Z *.jpg # 将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
```



## grep的使用

正则表达式的使用

```bash
# 找出以_id为结尾的非空字符串
grep -o -E '\S+_id' file.txt
```


## Redhat/Centos安装package
```bash
# 查询系统已安装的rpm包
rpm -qa

# 查询系统中一个已知的文件属于哪个rpm包
rpm -qf /${absolutePath}/file_name

# 查询已安装的软件包的相关文件的安装路径
rpm -ql ${packageName}

# 查询一个已安装软件包的信息
rpm -qi ${packageName}

# 查看已安装软件的配置文件
rpm -qc ${packageName}

# 查看已安装软件的文档的安装位置
rpm -qd ${packageName}

# 查看已安装软件所依赖的软件包及文件
rpm -qR ${packageName}

# 安装包
rpm -i ${packageName}.rpm
# 安装包并在安装过程中显示正在安装的文件信息(-v)及安装进度(-h)；
rpm -ivh ${packageName}.rpm

# 卸载软件
rpm -e --nodeps ${packageName}.rpm


# 查看系统未安装软件相关命令，也就是在-q后面加p
## 查看软件包的详细信息
rpm -qpi ${rpm-package}
## 查看软件包的文档所在的位置
rpm -qpd ${rpm-package}

```

### 参考
https://c.biancheng.net/view/817.html
