# 快速启动

## 启动NameServer
安装完RocketMQ包后，我们启动NameServer
```bash
nohup sh bin/mqnamesrv &
 
### 验证namesrv是否启动成功
tail -f ~/logs/rocketmqlogs/namesrv.log
### The Name Server boot success...

```


## 启动Broker+Proxy
```bash
### 先启动broker
nohup sh bin/mqbroker -n localhost:9876 --enable-proxy &

### 验证broker是否启动成功, 比如, broker的ip是192.168.1.2 然后名字是broker-a
tail -f ~/logs/rocketmqlogs/proxy.log 
### The broker[broker-a,192.169.1.2:10911] boot success...

```

## 工具测试消息收发
### 在机器1运行
```bash
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

### 在机器2运行
```bash
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```


## 关闭服务器
```bash
sh bin/mqshutdown broker
# The mqbroker(36695) is running...
# Send shutdown request to mqbroker with proxy enable OK(36695)

sh bin/mqshutdown namesrv
# The mqnamesrv(36664) is running...
# Send shutdown request to mqnamesrv(36664) OK
```



## 参考
https://rocketmq.apache.org/zh/docs/quickStart/01quickstart