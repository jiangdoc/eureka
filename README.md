Eureka
=====
## 核心概念
基于注册中心实现的服务发现，需要知道如下几个概念，或者说是问题。
**服务注册**：每个节点都需要主动把自己注册到中心里去。（用何种方式？？？http、tcp？）
**说明**：eureka它并不限制你用什么协议去通信，只不过它内置支持的是基于Rest的，你可以更改成RPC等
**健康检查监控**：若某个实例节点自己不可用了，注册中心如何知晓然后T除它呢？（新增了一个节点简单，它会主动注册上去嘛）
**客户端获取provider服务地址们**：每个节点要调用其它服务需要知道其具体的ip地址。如何获取？？？
**注册中心集群节点间信息共享**：注册中心有多个节点，多个节点之间数据如何sync？

## 包简介
- com.netflix.eureka.aws：与amazon AWS服务相关的类（我们一般用不着）
- com.netflix.eureka.cluster：与peer节点复制(replication)相关的功能，节点数据共享
- com.netflix.eureka.lease：即”租约”, 用来控制节点注册的生命周期(添加、清除、续约)
- com.netflix.eureka.registry：存储、查询服务注册信息
- com.netflix.eureka.resources：RESTful风格中的”R”, 即资源。相当于Spring MVC中的Controller
- com.netflix.eureka.transport：发送HTTP请求的客户端，如发送心跳（因为节点间赋复制需要它）

## 源码
### Eureka Client端动作
**服务注册Register**：client端主动向Server端注册。提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页…等
**服务续约Renew**：Client默认会每隔30秒发送一次心跳来续约。这样来告诉Server说我还活着，续约机制中，Server端默认在90秒没有收到Eureka客户的续约的话，就会将该实例从Server端T掉
**获取注册列表信息Fetch Registries**：Client端从Server获取注册表信息，并将其缓存在本地。缓存信息默认每30s更新一次（每次返回的和缓存的可能形同也可能不同，Client端自行处理从而发送不同的事件）
默认的情况下Client端使用压缩JSON格式来获取注册列表的信息（还支持xml格式）
**服务下线Cancel**：Client端在停机时（注意是正常停止）主动向Server端会发送取消请求，告诉Server端把自己这个实例T掉
### Eureka Server端动作

**提供服务注册**：
    - 服务注册接口：ApplicationResource.addInstance()
    - 服务注册信息：
**提供服务信息拉取**（查询）：
**提供服务管理**：接口客户端的cancle、心跳、续租renew等请求
**服务剔除Eviction**：在默认的情况下，当Eureka客户端连续90秒没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除。
**信息同步**：在集群中，每个Eureka Server同时也是Eureka Client。多个Server之间通过P2P复制的方式完成服务注册表的同步。
    同步规则：若有3个Eureka Server，Server1有新的服务信息时，同步到Server2后，Server2和Server3同步时，Server2不会把从Server1那里同步到的信息同步给Server3，只能由Server1自己同步给Server3
    
    
    
## 源码bolg
https://blog.csdn.net/f641385712/article/details/105610114
https://www.cnblogs.com/lay2017/p/11908715.html