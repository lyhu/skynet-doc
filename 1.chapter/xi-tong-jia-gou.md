# 1.2 系统架构

### 逻辑架构

![](../.gitbook/assets/001.png)



### 物理架构

![](../.gitbook/assets/002.png)

* AntServer： 平台服务器，上面可以部署多个平台服务，并对服务进行守护 
* AntWorker：服务节点，可以为基础服务或者引擎服务 

### 启动流程

![](../.gitbook/assets/003.png)

* xManager：集群管理界面，负责服务器的启动与关闭，服务的分配与去除
* ZooKeeper集群：集群数据存储节点，负责存储平台的相关数据





