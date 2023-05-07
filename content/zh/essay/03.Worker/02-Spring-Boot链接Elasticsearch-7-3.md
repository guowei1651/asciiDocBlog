---
title: "Spring-Boot链接Elasticsearch-7-3"
lead: "随笔"
date: 2023-04-22T12:52:56+08:00
lastmod: 2023-04-22T12:52:56+08:00
draft: false
images: []
menu:
  architecture:
    parent: "essay"
weight: 4000
toc: true
---

## 背景
在服务发展到一定阶段后，需要进行用户数据分析。在调研与设计用户行为分析系统的过程中需要从日志服务器Elasticsearch中解析用户行为日志。然后用日志记录用户的行为。在这个过程中发现对接Elasticsearch 7.3还是有一定的困难，所以记录下来并共享出来，为之后如果有遇到问题提供一种思路。

## 过程
使用Spring Boot最新代码以及spring-boot-starter-data-elasticsearch进行Elasticsearch的对接工作。在对接过程中一直在报：找不到可用的ES节点。所以，在调查中发现很多版本对应不上造成的问题。例如：使用Srping Boot的2.1.9.RELEASE版本，引入的spring-boot-starter-data-elasticsearch所附带的3.1.11.RELEASE的spring-data-elasticsearch包，而这个包应该是可以支持7.x的版本的。如下图所示：

![版本对应关系](images/essay/03-02-01.webp)

但是2.1.9的spring boot就是不能使用。使用的Spring Data Elasticsearch是3.1.11版本的，对于3.1.11这个版本支持到Elasticsearch协议的6.2.2的版本。而使用Elasticsearch 7.3的，支持的最低接口版本是6.8.0，所以，在2.1.19是不能对接Elasticsearch 7.3的。

查看原因应该是因为使用Elasticsearch Clients的6.8版本中有集群客户端（[PreBuiltTransportClient](https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.3/transport-client.html)）的调用，而在后续的版本中已经要删除这种方式了。所以，使用PreBuiltTransportClient就会造成问题。所以，升级Srping Boot到2.2.0版本应该不会有结果。不过还是升级到2.2.0试试。测试后是可以使用的。

## 结论
$\color{red}{如果需要使用Spring Boot链接Elasticsearch 7.3请使用2.2.0以上版本的Srping Boot}$。


## 参考
[Spring Data Elasticsearch - Reference Documentation](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RELEASE/reference/html/)
[Spring Data for Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)
[Doc Elasticsearch Clients](https://www.elastic.co/guide/en/elasticsearch/client/index.html)
