[微服务实践目录](https://www.jianshu.com/p/f3d5a02757f1)，可以参见连接。

# 背景

对于分布式应用来说最主要的是可管理性，因为微服务一个最主要的问题是使用外部复杂度替换内部复杂度，以减少内部复杂度对于人员、环境要求高的问题。

# 理论基础

### 调用链跟踪理论
[Google Dapper](http://bigbully.github.io/Dapper-translation/)

### Tracing和Monitor区别
监控和跟踪是有区别的。


# 实际要求

- **Server Topo图**
调用链
慢调用链查询
调用链记录
- **告警**
发送告警内容
- **全局管理**
全局开启和关闭Tracing
- **线上服务影响最小**
性能影响
侵入性影响

# 方案对比

zipkin,pinpoint,skywalking,cat,

# 参考

[什么才是“应用拓扑”？](https://www.jianshu.com/p/20dcc76aa9d9)
[分布式跟踪工具Pinpoint初探](https://www.imooc.com/article/29232)

[调用链选型之Zipkin，Pinpoint，SkyWalking，CAT](https://www.jianshu.com/p/0fbbf99a236e)
[全链路监控（一）：方案概述与比较](https://juejin.im/post/5a7a9e0af265da4e914b46f1)

[Pinpoint安装详解](https://www.jianshu.com/p/6d7ce28bb74d)
[开源APM监控Pinpoint的快速部署和使用](https://www.jianshu.com/p/ac1219c67e3c)
[使用pinpoint进行SpringCloud服务链监控](https://www.jianshu.com/p/5a6dc609acea)
[APM监控工具之Pinpoint初探](https://www.jianshu.com/p/6d1738db962c)
[APM监控工具之Pinpoint初探](http://laciagin.me/2018/04/10/APM%E7%9B%91%E6%8E%A7%E5%B7%A5%E5%85%B7%E4%B9%8BPinpoint/)

[pinpoint](https://github.com/naver/pinpoint)
