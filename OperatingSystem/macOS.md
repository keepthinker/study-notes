## Command

### 查看服务

```bash
# 查看全部服务
sudo launchctl list
# 查看docker服务
launchctl list | grep -i docker
# 关闭docker服务
launchctl stop application.com.docker.docker.22511269.22511645
# 启动docker服务
open /Applications/Docker.app/ 
# 查看网络端口和进程
lsof -i tcp -n -P | grep LISTEN
netstat -anv | grep LISTEN
```


