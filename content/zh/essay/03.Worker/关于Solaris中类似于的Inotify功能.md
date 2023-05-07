---
title: "关于Solaris中类似于的Inotify功能"
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
当年的巨无霸一去不复返了啊。在SUN收购Mysql之后，我们还在讨论SUN已经是一个全产业链的超级大公司了。那时的SUN有语言JAVA，有操作系统Solaris，有数据库Mysql，有IDE，有硬件。

## 原文：
可以使用[http://blogs.sun.com/praks/entry/file_events_notification](http://blogs.sun.com/praks/entry/file_events_notification)中介绍的port方法，在用户态监视系统中某个文件(大家都知道*nix中的文件，并不一定真的是文件，可以是目录等等一些东西)。该机制可以监控的事件有：

     Watchable events:
```C
FILE_ACCESS             /* Monitored file/directory was accessed */
FILE_MODIFIED        /* Monitored file/directory was modified */
FILE_ATTRIB             /* Monitored file/directory's ATTRIB was changed */
FILE_NOFOLLOW    /* flag to indicate not to follow symbolic links */
```
     Exception events - cannot be filtered:
```C
FILE_DELETE                 /* Monitored file/directory was deleted */
FILE_RENAME_TO        /* Monitored file/directory was renamed */
FILE_RENAME_FROM /* Monitored file/directory was renamed */
UNMOUNTED                /* Monitored file system got unmounted */
MOUNTEDOVER          /* Monitored file/directory was mounted on */
```
这些事件类型定义在`sys/port.h`中，在原文中可以看到这个机制不止可以监视文件事件。在使用文件事件通知(File Events Notification)时，像socket变成一样现需要一个socket。在FEN中需要的port，可以用`port_create`函数创建port。创建port后可以使用函数`port_associate`来注册该port上要监视的事件源的类型，再将你的文件的现在的状态填到结构体file_obj_t中传给该函数。含有就是你要系统通知你的事件(不过我在测试的时候,只设置`FILE_ACCESS|FILE_MODIFIED|FILE_ATTRIB`其他事件一样会通知你)。用户数据会在调用`port_get`的返回参数`port_event_t`中的`portev_object`中填写。

所有工作完成了，可以直接调用函数port_get等待事件发生了。

哎！自从Oracle接手SUN之后，文档系统就可以说根本找不到SUN的信息了。够狠。

另附：函数声明
```C
#include <port.h> 
int port_create(void); /* 该函数用创建port */
/* 下面两个函数一个注册一个注册文件事件通知 */
int port_associate(int port, int source, uintptr_t object, int events, void *user);
int port_dissociate(int port, int source, uintptr_t object);
/* 在注册文件事件通知之后，需要调用该函数来得到事件，这两个函数会阻塞 */
int port_get(int port, port_event_t *pe, const timespec_t *timeout);
int port_getn(int port, port_event_t list[], uint_t max, uint_t *nget, const timespec_t *timeout);
```
