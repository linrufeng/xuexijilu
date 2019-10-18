---
title: mongodb学习
date: 2018-09-04 10:10:03
tags:
---
### 安装mongodb

```
sudo brew install mongodb
```

### 运行mongodb

1、首先我们创建一个数据库存储目录 /data/db：

```
sudo mkdir -p /data/db
```

启动 mongodb
```
sudo mongod
```
此时，可以通过http访问该数据库，mongodb使用了27017端口，因此在浏览器中打开http://localhost:27017/。
出现如下提示即说明连接成功了。

It looks like you are trying to access MongoDB over HTTP on the native driver port.

### 链接mongodb

```
$ sudo mongo
```
这时如果顺利
```
ZBMAC-C02VX135H:/ lilinsen$ sudo mongo
MongoDB shell version v4.0.1
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 4.0.1
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] ** WARNING: This server is bound to localhost.
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          Remote systems will be unable to connect to this server. 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          Start the server with --bind_ip <address> to specify which IP 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          addresses it should serve responses from, or with --bind_ip_all to
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          bind to all interfaces. If this behavior is desired, start the
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] **          server with --bind_ip 127.0.0.1 to disable this warning.
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] 
2018-09-04T09:28:58.205+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
```
这时就链接到了db
我们可以对数据库进行操作了
可以试着尝试如下的命令
```
> db
test
> use huiwei
switched to db huiwei
> db
huiwei
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> db.runoob.insert({"name"："李林森"})
2018-09-04T10:01:39.653+0800 E QUERY    [js] SyntaxError: illegal character @(shell):1:24
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> db.runoob.insert({"name":"李林森"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
huiwei  0.000GB
local   0.000GB
> 
```


