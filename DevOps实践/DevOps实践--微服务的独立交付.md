# 背景

回到最初马丁 • 福勒对[微服务](https://martinfowler.com/articles/microservices.html)的定义：微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务，每个服务在其独立的进程中，服务间采用轻量级的通信机制互相沟通（通常是基于HTTP协议的RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被$\color{red}{独立的部署}$到生产环境、类生产环境等。

而我们在研发过程中在很多原因的作用下让调用链形成：链式调用，聚合器模式。在这种模式下会发现我们的微服务慢慢形成了一张蜘蛛网。像netfilx的服务间关系一样：
![netfilx调用链](https://upload-images.jianshu.io/upload_images/2454595-67901781965d8260.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在这种情况下服务间关系不可能在任何一个人的头脑中形成完整概念。所以在此情况下怎样升级服务，独立部署服务就成了一个大问题。

# 什么影响独立部署

从上图的蜘蛛网图中，不能保证服务可以独立演进就更不用提独立部署了。而从服务治理和业务发展的角度来看

- #### 卡方测试
用户对新版本的满意度

- #### 接口版本兼容
接口的v1版和v2版之间的切换

- #### 功能开关
对哪些用户开放新功能？
使用哪些用户进行新功能的验证？

- #### 演进式过程
无主体设计的情况下，使用业务驱动的方式对系统进行研发。

# 独立交付的层面

- #### 代码实现层面
服务循环依赖的问题

- ### 部署层面

不使用服务注册中心是不是能起来。
不依赖于其他服务的时候能否正常的提供服务。

- #### 业务层面
是否可以将业务拆离出来独立的部署，独立的演进的能力。相当于原先一个商城的业务，将其拆解成 对账业务 和  商城业务？

- #### 

# 解决方案


契约测试
混沌工程

# 总结


# 参考：
[阿里巴巴是如何管理测试环境的？](https://www.infoq.cn/article/AG6gBTD9w7j4sGjRDrWK)


[Microservices](https://martinfowler.com/articles/microservices.html)
[微服务](http://blog.cuicc.com/blog/2015/07/22/microservices/)
[你的微服务敢独立交付么？](https://www.jianshu.com/p/625476437c22)
[第十二届「中国持续交付大会」-北京站](https://www.itdks.com/Home/Act/apply?id=1676)
[微服务架构体系的深度治理](https://www.infoq.cn/article/q65dDiRTdSbF*E6Ki2P4)
[Netflix 是这样构建微服务技术架构的](https://zhuanlan.zhihu.com/p/31469970)
[微服务治理实践 | 金丝雀发布](https://mp.weixin.qq.com/s/T6mn1Pydhv6zRWUTzihf9A)
