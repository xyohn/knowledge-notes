# RocketMQ

1. NameServer

   1. 特性

      1. 无状态，互不沟通
      2. 保存集群内所有节点的路由信息

   2. 数据结构

      > 路由信息保存在内存中,并且没有持久化
      >
      >  org.apache.rocketmq.namesrv.routeinfo.RouteInfoManager

      * topicQueueTable

        > 主题和队列信息，其中每个队列信息对应的类 QueueData 中，还保存了 brokerName
        >
        > 这个 brokerName 并不真正是某个 Broker 的物理地址，它对应的一组 Broker 节点，包括一个主节点和若干个从节点

      * brokerAddrTable

        >  保存了集群中每个 brokerName 对应 Broker 信息，每个 Broker 信息用一个 BrokerData对象
        >
        > BrokerData 中保存了集群名称 cluster，brokerName 和一个保存 Broker 物理地址的 Map：brokerAddrs，它的 Key 是 BrokerID，Value 就是这个 BrokerID 对应的 Broker 的物理地址

      * clusterAddrTable

        > 保存的是集群名称与 BrokerName 的对应关系

      * brokerLiveTable

        > 保存了每个 Broker 当前的动态信息，包括心跳更新时间，路由数据版本等等

      * filterServerTable

        > 保存了每个 Broker 对应的消息过滤服务的地址，用于服务端消息过滤

   3. 核心实现

      1. Broker 注册路由

         核心逻辑 #registerBroker

         > 使用了ReadWriteLock实现对数据结构加写锁，依次对比并更新 clusterAddrTable、brokerAddrTable、topicQueueTable、brokerLiveTable 和 filterServerTable

      2. 客户端寻址Broker

         核心逻辑 #pickupTopicRouteData

         * 使用了ReadWriteLock实现对数据结构加读锁
         * 根据Topic名称查询topicQueueTable获取BrokerName和Queue的信息
         * 根据BrokerName查询brokerAddrTable获取所有的Broker
         * 根据BorkerAddr查询filterServerList获取过滤的地址

         

