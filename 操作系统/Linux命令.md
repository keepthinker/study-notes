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
systemctl start  firewalld # 启动

systemctl status firewalld # 或者 firewall-cmd --state 查看状态

systemctl disable firewalld # 停止

systemctl stop firewalld  # 禁用

## 端口管理
打开443/TCP端口

firewall-cmd --add-port=443/tcp

永久打开3690/TCP端口

firewall-cmd --permanent --add-port=3690/tcp

永久关闭端口

firewall-cmd --remove-port=80/tcp --permanent 

永久打开端口好像需要reload一下，临时打开好像不用，如果用了reload临时打开的端口就失效了
其它服务也可能是这样的，这个没有测试

firewall-cmd --reload

查看防火墙，添加的端口也可以看到

firewall-cmd --list-all

## 参考文献
[Linux firewall-cmd 命令详解](https://blog.csdn.net/GMingZhou/article/details/78090963)