---
title: "Jenkins的环境管理讨论"
lead: "02 未划分"
date: 2023-04-22T12:52:56+08:00
lastmod: 2023-04-22T12:52:56+08:00
draft: false
images: []
menu:
  devops:
    parent: "devops"
weight: 4000
toc: true
---

## 背景

在经过公司内部自研持续交付环境，再到使用很长时间的Jenkins之后。对于持续交付工具的一些特点有一定认识。在为软件开发流程设计持续交付过程中需要考虑到很多方面的内容。一般通用的持续集成系统中都会设计到以下几个部分：
1. **环境管理**。在持续集成环境中包括很多类型的环境。例如：持续交付运行环境，编译环境，存储环境，发布环境，被测环境，测试环境等等。之前在通信行业进行持续交付环境构建时，考虑到硬件资源以及硬件型号的问题对于不同的硬件型号需要运行不同的构建与测试过程。
2. **Pipeline流程**。为持续交付提供流程、步骤的支持，可以使用原子操作、组合操作进行组合形成一套Pipeline流程。最终满足持续交付的过程需要。
3. **原子操作**。针对通信行业中的交付过程，可能涉及到不同的网元。对于网元间版本控制与接口对接的以及测试要求需要考虑。所以，原子操作可以拆成对网元的操作。对于互联网应用中的微服务体系的操作，可以定义一套编译，构建，发布是原子操作。每个微服务都是由这些原子操作的过程组成，然后这些微服务的统一管理的pipeline又可以是一个更上层的流程。
4. **结果展示**。各项结果都需要有好的展示出来。代码扫描结果，编译结果，流程结果，原子操作结果等等都是需要展示出来的。

综上，要构建一套持续交付环境需要对各个层级有比较清晰的认识之后再做环境构建时是比较好的。本篇文章着重讨论**环境管理**部分的内容。

## 环境管理发展

环境管理发展描述从比较单一的持续构建环境一路发展为环境即代码环境管理方式。这个过程从某个侧面可以认为是IT业界对环境管理过程的认知升级的过程。一步步的从手动管理环境到自动化调度、控制环境的过程。

- ### 1. 直接使用Jenkins服务器做为构建环境
在刚开始接触Jenkins时，一般都会使用Jenkins Server作为我们所有动作执行的地方。在Jenkins Server上执行代码Checkout，编译，依赖管理，制品管理。慢慢的随着要处理的微服务个数，要存储的制品、要做的频率不断的提升会逐渐的觉得只使用Jenkins Server完成这些工作比较吃力。想办法逐渐的把各项工作分散到不同的机器上完成。可能通过ssh，ansible，chef等工具完成。

- ### 2. 使用Jenkins的Node管理作为构建环境
在用ssh一段时间之后，发现ssh的服务器只能通过代码进行管理。在同时执行一个Job的时候回发生环境冲突的问题。ssh不能进行环境调度的工作，只能将Job和主机绑定。从这个时候开始借助Jenkins Agent来进行辅助服务器的管理工作。使用Jenkins Agent可以对服务器进行包活，任务调度的管理。

- ### 3. 使用Docker作为Node进行环境管理
在经过对Job的调度工作时候，慢慢的发现每个Agent都是一个持续交付的环境。每个Agent上跑着所有的任务，任务都在Agent上存储一部分Job数据。Job间可能会造成排队与存储干扰的问题。在持续交付环境管理中已经投入了机器的情况下，还是会遇到一些环境的一些问题。就需要考虑怎样处理这些问题。正好Docker第一个功能就是做环境隔离的工作。用Docker将不同的环境隔离出来，例如：专门做Node编译的，专门做Java编译的，专门做C#编译的，专门做发布的。都可以独立的隔离出来，这样可以按照持续交付的步骤进行环境的隔离，解决排队和干扰的问题。

- ### 4. 使用一次性Docker容器作为Node的环境管理
慢慢的固定的Docker容器去做工作，会发现Docker很多时候都是空闲的。所以，就想怎样提高资源的利用率。开始使用一次性Docker解决问题，最终可以做到随建随走，随用随走。

- ### 5. 使用环境即代码的方式管理环境
在Docker的仓库，Docker与物理机之间的关系等在发展到一定阶段之后也是需要进行大量的管理的。这个时候Docker的调度，Docker的恢复，Docker的监控，Docker的镜像等都需要进行管理。在这个使用控制环境如果还是使用某种二进制包+说明文档的方式进行管理就不能很好的完成工作了。这个时候就明显的需要对环境进行版本化管理，对环境进行版本化管理可以很快的复制一套相同的环境做其他用途。

## 实现方式

从上面的环境管理发展可以总结出环境管理的几个要点任务：
- 环境包活
- 任务调度
- 环境隔离
- 资源利用率
- 环境管理版本化

针对每一个发展阶段都有它自己的技术特点。每个阶段的技术主要内容有：
1. Jenksin、Gitlab、TeamCity、Travis CI
2. Agent
3. Docker
4. Swarm，K8s，ansible

这里给出来一个Jenkins上使用一次性Docker的一个例子：
1. 环境构建
![Jenkins环境](images/devops/99-01-01.webp)
2. 配置Pipeline
```
pipeline {
    agent {
        docker {
            image 'maven:3.3.3'
            args  '-v /tmp:/tmp -v $HOME/.m2:/root/.m2 -v /opt/docker/ci/pmd/p3c:/opt/p3c -v p3c_result:/opt/result'
        }
    }
	options {
		timeout(time: 1, unit: 'HOURS')
	}
    stages {
        stage("init workspace") {
            steps {
                cleanWs()
            }
        }
        stage("checkout code"){
            steps {
                echo "checkout code"
                checkout([$class: 'GitSCM', branches: [[name: '*/dev']], doGenerateSubmoduleConfigurations: false, 
                    extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'wales_kuo', url: git_url]]])
            }
        }
        stage("scan code"){
            steps {
                echo "scan code"
                sh '''
				export p3c_jar_path="/opt/p3c/p3c-pmd-2.0.0.jar:/opt/p3c/lib/antlr4-runtime-4.7.jar:/opt/p3c/lib/asm-7.1.jar:/opt/p3c/lib/commons-io-2.6.jar"
				export p3c_jar_path=$p3c_jar_path:"/opt/p3c/lib/commons-lang3-3.8.1.jar:/opt/p3c/lib/gson-2.8.5.jar:/opt/p3c/lib/javacc-5.0.jar"
				export p3c_jar_path=$p3c_jar_path:"/opt/p3c/lib/javax.annotation-api-1.3.2.jar:/opt/p3c/lib/jcommander-1.72.jar:/opt/p3c/lib/pmd-core-6.15.0.jar:"
				export p3c_jar_path=$p3c_jar_path:"/opt/p3c/lib/pmd-java-6.15.0.jar:/opt/p3c/lib/pmd-vm-6.15.0.jar:/opt/p3c/lib/saxon-9.1.0.8.jar"
				
				export p3c_rulesets="rulesets/java/ali-comment.xml,rulesets/java/ali-concurrent.xml,rulesets/java/ali-constant.xml,rulesets/java/ali-exception.xml"
				export p3c_rulesets=$p3c_rulesets,"rulesets/java/ali-flowcontrol.xml,rulesets/java/ali-naming.xml,rulesets/java/ali-oop.xml,rulesets/java/ali-orm.xml"
				export p3c_rulesets=$p3c_rulesets,"rulesets/java/ali-other.xml,rulesets/java/ali-set.xml,rulesets/vm/ali-other.xml"

				export CWD=$(pwd)
				export result_path=/opt/result/'''+BUILD_NUMBER+'''
				mkdir -p $result_path
				java -cp "$p3c_jar_path" net.sourceforge.pmd.PMD -R $p3c_rulesets -d $CWD -f yahtml -P outputDir=$result_path | exit 0'''
            }
        }
        stage("run test"){
            steps {
                echo "run test"
            }
        }
        stage("build service"){
            steps {
                echo "build"
                // sh 'mvn clean package'
            }
        }
        stage("publish service"){
            steps {
                echo "public"
                
            }
        }
    }
}
```
3. 完成。


## 参考：
[持续交付](https://book.douban.com/subject/6862062/)
[持续交付2.0](https://book.douban.com/subject/30419555/)
[DevOps实践指南](https://book.douban.com/subject/30186150/)
