## 端口转发

```batch
# 删除端口绑定
netsh interface portproxy delete v4tov4 listenport=10022 listenaddress=* protocol=tcp

# 显示所有代理端口设置
netsh interface portproxy show all

# 重置代理设置
netsh interface portproxy reset

# 添加一个端口
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=22 connectaddress=192.168.44.155 connectport=22

# 参考https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc776297(v=ws.10)?redirectedfrom=MSDN#BKMK_1
```

## win10 linux 子系统 wsl2实现ip自动转发

[win10 linux 子系统 wsl2实现ip自动转发_nvd11的博客-CSDN博客_wsl2 端口转发](https://blog.csdn.net/nvd11/article/details/128047248)

## 进入ubuntu

直接输入命令 "bash"

## Powershell命令

```bash
tasklist /fi  "imagename eq java.exe"

tasklist | findstr "java"

# 查看2001端口 导出到c盘
netstat -aont | findstr "2001" > c:\2001.log
```