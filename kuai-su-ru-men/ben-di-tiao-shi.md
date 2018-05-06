# 2.3 本地调试

以Eclipse开发工具为例:

#### 启动工程：

${对应的插件工程}

> tuling-ast

#### 入口类：

```text
skynet.cloud.App
```

#### 启动参数：

```text
--type=worker
--address=127.0.0.1
--server.port=2236
--name=rest-app-v10@sample
--eureka.client.serviceUrl.defaultZone=http://192.168.83.206:7070/eureka
```

#### JVM参数

```text
-Dskynet.zookeeper.server_list=192.168.83.206:218
-Dskynet.zookeeper.cluster-name=skynet
-Dskynet.zookeeper.session-timeout=20000
-Dskynet.zookeeper.connection-timeout=10000
```

> * type 必选
> * address 是本机的IP
> * server.port 对外服务端口
> * name 是必选，是对应具体业务服务的 Action全名称，格式一般为：{action名称}@{plugin名称}



