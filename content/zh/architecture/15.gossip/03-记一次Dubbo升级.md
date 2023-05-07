---
title: "03 记一次Dubbo升级"
lead: "15.闲聊"
date: 2023-04-22T12:52:56+08:00
lastmod: 2023-04-22T12:52:56+08:00
draft: false
images: []
---

## 背景

公司有一批15年左右实现的中台服务，当时使用的是JDK1.7+Gradle3.7实现的。这套微服务现在还是公司的业务主力，经过5年做左右的积累这些服务已经复杂的不可修改了。基于此公司准备对这批服务进行重构。重构的第一步肯定是进行技术升级。

## 过程

在进行技术升级的时候，公司对微服务进行技术升级的时候规划了几个步骤：
1. 升级JDK。将JDK从1.7升级到1.8。
2. 升级Dubbo。将Dubbo从2.5.3升级到2.7.3
3. 升级到Spring Cloud。
4. 使用容器管理服务。
5. 上云原生应用。

做好计划之后，进行具体的升级工作：
- **进行JDK的升级**
修改Gradle的build文件中的Jdk版本到
```gradle
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
```
然后进行编译，编译完发不到测试环境都挺顺利的。在测试环境上运行起来之后就发现：
```log
java.lang.ClassNotFoundException: javassist.ClassPath
```
调查发现Dubbo2.5.3使用的org.javassist的包版本是3.15.0-GA。而这个版本在JDK1.8上不可运行。这出发了一个不可接受的问题，Dubbo2.5.3不可以在JDK1.8上运行。一般开源的服务都会对不同的JDK版本进行适配与支持，现在Dubbo2.5.3已经突破了底线了。

既然发现了这个问题，那就把第二步要做的升级Dubbo的事情一起做了吧。对Dubbo进行升级，升级到2.7.3版本：
1. 修改gradle的build文件中的Dubbo到2.7.3:
```
com.alibaba:dubbo:2.5.3   -> org.apache.dubbo:dubbo:2.7.3
```
2. 修改Dubbo的xml配置文件中
```
http://code.alibabatech.com/schema/dubbo -> http://dubbo.apache.org/schema/dubbo

http://code.alibabatech.com/schema/dubbo/dubbo.xsd -> http://dubbo.apache.org/schema/dubbo/dubbo.xsd
```
3. 修改Dubbo配置中的。因为logger从下升级的话可能不可使用，所以需要使用制定logger。
```
<dubbo:application name="uc" logger="slf4j"/>
```
4. 修改代码中使用dubbo的代码
```java
com.alibaba.dubbo.common.utils.StringUtils
com.alibaba.dubbo.common.utils.CollectionUtils
```
到此就可以跑起来了。然后升级的服务和其他的服务之间的rpc通信也没有断。所以在这个过程中可以看到dubbo版本之间是可以兼容的。

在跑了两天之后发现一个问题，在做一个业务的时候无缘无故的发生了rollback问题。所有的异常在代码中都已经try-catch了，为什么会发生rollback问题呀。而且还是在两个服务通过rpc之间传递了rollback异常：
```
[DubboServerHandler-0.0.0.0:20880-thread-305] [ERROR] [o.a.d.r.f.ExceptionFilter$ExceptionListener] - [ [DUBBO] Got unchecked and undeclared exception which called by 0.0.0.0. service: cn.thinkjoy.uc.service.business.IUserService, method: saveUser, exception: org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only, dubbo version: 2.7.3, current host: 0.0.0.0]
org.springframework.transaction.UnexpectedRollbackException: Transaction rolled back because it has been marked as rollback-only
        at org.springframework.transaction.support.AbstractPlatformTransactionManager.commit(AbstractPlatformTransactionManager.java:720) ~[spring-tx-4.1.0.RELEASE.jar:4.1.0.RELEASE]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.commitTransactionAfterReturning(TransactionAspectSupport.java:496) ~[spring-tx-4.1.0.RELEASE.jar:4.1.0.RELEASE]
        at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:276) ~[spring-tx-4.1.0.RELEASE.jar:4.1.0.RELEASE]
        at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:95) ~[spring-tx-4.1.0.RELEASE.jar:4.1.0.RELEASE]
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179) ~[spring-aop-4.3.16.RELEASE.jar:4.3.16.RELEASE]
        at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:213) ~[spring-aop-4.3.16.RELEASE.jar:4.3.16.RELEASE]
        at com.sun.proxy.$Proxy70.saveUser(Unknown Source) ~[na:na]
```
无奈只能去查看Spring Tx的代码，发现与异常对应的点为UnexpectedRollbackException的位置：
```java
	@Override
	public final void commit(TransactionStatus status) throws TransactionException {
		if (status.isCompleted()) {
			throw new IllegalTransactionStateException(
					"Transaction is already completed - do not call commit or rollback more than once per transaction");
		}

		DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
		if (defStatus.isLocalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Transactional code has requested rollback");
			}
			processRollback(defStatus);
			return;
		}
		if (!shouldCommitOnGlobalRollbackOnly() && defStatus.isGlobalRollbackOnly()) {
			if (defStatus.isDebug()) {
				logger.debug("Global transaction is marked as rollback-only but transactional code requested commit");
			}
			processRollback(defStatus);
			// Throw UnexpectedRollbackException only at outermost transaction boundary
			// or if explicitly asked to.
			if (status.isNewTransaction() || isFailEarlyOnGlobalRollbackOnly()) {
				throw new UnexpectedRollbackException(
						"Transaction rolled back because it has been marked as rollback-only");
			}
			return;
		}

		processCommit(defStatus);
	}
```
那就是只能说记录了在业务流程中发生了异常，那就需要进一步查看上面打印出来的异常。rollback之前还是有一个异常的：
```
org.apache.dubbo.rpc.RpcException: No provider available from registry 127.0.0.1:2181 for service xx.XXX.xxx on consumer 0.0.0.0 use dubbo version 2.7.3, please check status of providers(disabled, not registered or in blacklist).
```
查看代码发现：
```java
        try {
            XXX.xxxx(xxx, xxx);
        } catch (RpcException e) {
            LOGGER.error("调用异常" + xxx, e);
        }
```
为啥这个异常捕获没有捕获到。调查发现：RpcException的import是com.alibaba.dubbo.rpc.RpcException。但现在使用的是org.apache的dubbo。所以获取到，导致异常从这段代码的方法中抛出到外部。修改catch中的捕获变为Throwable就解决了问题。

## 总结
Dubbo对于底层运行环境的依赖过于的强，导致升级jdk的时候必须升级Dubbo版本。如果不同的Dubbo版本之前序列化和反序列化出现问题，那就可以不用玩了。如果利用一些Dubbo的特性在过程中传输一些上下文信息，那也不要玩了。

升级一个jdk牵扯出来这么多dubbo的事情为啥感觉这么不能接受呢。因为同样的环境下我们还用了spring 4.1.0，然后升级完jdk之后spring没有任何反应者多爽呢。
