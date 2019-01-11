# dubbo源码解析 #
# 图解 #
![](https://i.imgur.com/PDyQelw.png)
![](https://i.imgur.com/0hhdKBx.png)

图例说明：
图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供
方使用的接口，位于中轴线上的为双方都用到的接口。
图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依
赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，
其它各层均为 SPI。
图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的
实现类。
图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即
运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线
上的文字为调用的方法。
# 关系说明 #
在 RPC 中，Protocol 是核心层，也就是只要有 Protocol + Invoker + Exporter
就可以完成非透明的 RPC 调用，然后在 Invoker 的主过程上 Filter 拦截点。
图中的 Consumer 和 Provider 是抽象概念，只是想让看图者更直观的了解哪
些类分属于客户端与服务器端，不用 Client 和 Server 的原因是 Dubbo 在很多
场景下都使用 Provider, Consumer, Registry, Monitor 划分逻辑拓普节点，保持
统一概念。
而 Cluster 是外围概念，所以 Cluster 的目的是将多个 Invoker 伪装成一个
Invoker，这样其它人只要关注 Protocol 层 Invoker 即可，加上 Cluster 或者去
掉 Cluster 对其它层都不会造成影响，因为只有一个提供者时，是不需要
Cluster 的。
2 框架设计
7
Proxy 层封装了所有接口的透明化代理，而在其它层都以 Invoker 为中心，只
有到了暴露给用户使用时，才用 Proxy 将 Invoker 转成接口，或将接口实现转
成 Invoker，也就是去掉 Proxy 层 RPC 是可以 Run 的，只是不那么透明，不
那么看起来像调本地服务一样调远程服务。
而 Remoting 实现是 Dubbo 协议的实现，如果你选择 RMI 协议，整个
Remoting 都不会用上，Remoting 内部再划为 Transport 传输层和 Exchange
信息交换层，Transport 层只负责单向消息传输，是对 Mina, Netty, Grizzly 的
抽象，它也可以扩展 UDP 传输，而 Exchange 层是在传输层之上封装了
Request-Response 语义。
Registry 和 Monitor 实际上不算一层，而是一个独立的节点，只是为了全局概
览，用层的方式画在一起。