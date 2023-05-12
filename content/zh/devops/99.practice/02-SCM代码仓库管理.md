---
title: "SCM代码仓库管理"
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
人在不同的阶段对于同一件事会有不同的理解。而且每个人都来说个人方向选择、精力与经历都左右着对同一件事情的理解。所以，
> 三人行，则必有我师。是故弟子不必不如师，师不必贤于弟子，闻道有先后，术业有专攻，如是而已。 --《师说》韩愈

抱着开放的心态学习每一个比你优秀的人的优点。就是我们这些软件从业者应该做的。抱着为软件业做一份贡献的执着的心情，持续得在这个行业里钻研和研究。

在软件公司之间最大的差别是什么？最大的差别在于公司的文化。不管是工程师文化、敏捷文化、精益MVP文化、商业目标文化，都是为了公司更好的发展以及在行业内与其他竞品进行竞争。这个文化最大的指导意义是指导软件团队的可靠性与效率。

对于完整的DevOps体系来说普元的王海龙的这张图可以非常完整的说明整个DevOps中所有的方面的内容：

在DevOpsMaster上
![DevOpsMaster白皮书](images/devops/99-02-01.webp)

![DevOps体系](images/devops/99-02-02.webp)


还有DevOps工具集的集合：
![](images/devops/99-02-03.webp)

## 介绍

对于公司内部软件公司内部比较重要的几件事：业务方案，软件过程，技术方案，基础设施。

对于SCM(Source Contronl Management)来说应该在DevOps中的位置可以在上面看到，SCM是用来支撑DevOps的基础设施。它为DevOps流程提供最基础的原材料（代码）的管理工作。所以在SCM之上我们应该追求的更多，最求满足各个层面的要求。要支持业务方案的变化，要支持软件过程，要支持技术方案管理，要支持基础设施的配置管理。并且要支持团队的工作量化，质量量化，效率改进等内容。

![DevOps目标](images/devops/99-02-04.webp)

## 技术选型
在软件业界有各个方面可以提供代码托管服务，并且很多代码托管系统都可以与云设备进行深度的融合工作。深度融合后的代码托管服务在加上《一切即代码》理念，就可以管理代码、依赖、编译、配置、流程、设备等。下面对代码托管服务进行技术选型评比：

![SCM对比](images/devops/99-02-05.webp)
对于可以私有化部署，并且支持更好的公司协作。费用又较低的只有Gitlab可以满足要求。GitHub，BitBucket可以私有化部署，可以支持很多功能，有完善的工具链。但Gitlab的开源协议是MIT License。并且GitLab在企业内部的使用率也很高。

## 技术实践
直接使用Docker镜像进行安装。使用[omnibus-gitlab](https://gitlab.com/gitlab-org/omnibus-gitlab/tree/master)的[docker-compose.yml](https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/docker/docker-compose.yml "docker-compose.yml")直接启动Docker即可。

- #### Git协议端口修改
中间需要注意的是，因为Git使用了SSH的端口，需要将SSH的端口修改为其他端口。将TCP22留给SSH使用。

- #### 备份工作
配置备份与镜像方式。在每个Git仓库中进行Mirroring的配置即可，但是需要每个项目独立配置。在使用过程中因为使用invalid multibyte char (US-ASCII),在网上搜索后发现是因为中文问题不能处理导致。所以重新进行Docker build：
```
FROM gitlab/gitlab-ce
MAINTAINER GitLab Inc. <support@gitlab.com>

SHELL ["/bin/sh", "-c"],

# Install required packages
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive yum groupinstall -y chinese-support
    && rm -rf /var/lib/apt/lists/* \

# Default to supporting utf-8
ENV LANG=en_US.UTF-8
```
配置并设置好语言后发现仍不可用。已经不报错了，但是同步一直没有结束。另外在检查Gitlab API时没有关于调用Mirror同步的API。所以，这里不考虑这种方式进行同步。

使用最原始的同步方式Git命令的方式进行同步。在使用Python(Pexpect)进行命令执行进行大量的Git仓库同步。
```python
#-*- coding: UTF-8 -*
import os
import sys
import base64
import pexpect
import traceback
import subprocess

mirror_git_base_addr='http://外网备份地址/'
master_git_base_addr='http://内网地址/'

def setOriginRemote(item, url):
    print 'set remote', item, url
    command = 'git remote set-url origin '+url
    try:
        popen = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True, universal_newlines=True)    
        result = None
        while result is None:
            result = popen.poll()
        print "set url status", result
    except:
        pass
        traceback.print_exc()

def gitPull(item):
    print 'git pull', item
    git_addr = master_git_base_addr+item+".git"
    setOriginRemote(item, git_addr)
    command = 'git fetch --tags --progress ' + git_addr + ' +refs/heads/*:refs/remotes/origin/*'
    print command
    child = pexpect.spawn(command)
    try:
        print "child logfile"
        child.logfile = sys.stdout
        print "set username"
        child.expect('Username')
        child.sendline('用户名')
        print "set password"
        child.expect('Password')
        child.sendline('密码')
	
        child.expect(pexpect.EOF, timeout=None)
    except:
        traceback.print_exc()
        pass

def gitPush(item):
    git_addr = mirror_git_base_addr+item+".git"
    setOriginRemote(item, git_addr)
    command = "git push origin 'refs/remotes/origin/*:refs/heads/*'"
    child = pexpect.spawn(command)
    try:
        child.logfile = sys.stdout
        child.expect('Username')
        child.sendline('用户名')
        child.expect('Password')
        child.sendline('密码')
        child.expect(pexpect.EOF, timeout=None)
    except:
        #print(str(child))
        # print ''
        traceback.print_exc()
        # print ''
        pass

def syncGitToMirror(item):
    print 'syncGitToMirror', item
    gitPull(item)
    gitPush(item)

if __name__ == '__main__':
    currentDir = os.getcwd()
    
    projects = ['Requirement', 'Business', 'Deploy']
	
    for item in projects:
        os.chdir(item)
        syncGitToMirror(item)
        os.chdir(currentDir)
```


- #### 本地备份
可以选择多种方式进行备份与恢复的工作。这里在介绍一下本地备份的方式：
1. 编辑/etc/gitlab/gitlab.rb
```
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/data/gitlab/backups"    //gitlab备份目录
gitlab_rails['backup_archive_permissions'] = 0644       //生成的备份文件权限
gitlab_rails['backup_keep_time'] = 7776000              //备份保留天数为3个月（即90天，这里是7776000秒）
```
2. 执行备份命令
```
gitlab-rake gitlab:backup:create
```
3. 设置定时备份（cron）
```
0 0,6,12,18 * * * /bin/bash -x /data/gitlab/backups/gitlab_backup.sh > /dev/null 2>&1
```
4. 恢复备份
```bash
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-ctl status
# 文件名前半部，如下所示
gitlab-rake gitlab:backup:restore BACKUP=1568640014_2019_09_16_12.1.4
```

## 总结
Gitlab在公司内部使用基本上已经替代了SVN的管理。并且提供了Git的很多敏捷功能支持。不过在这个过程中需要考虑怎样提供部署级的高可用和高性能。需要有自行处理。这个时候就可以很好的设计与完善系统。

每一项技术都有自己的优缺点，需要选择技术的优点。规避技术缺陷，这个时候需要有能力平衡这些有缺点。才能形成完善的技术解决方案。

## 参考
[大话微服务与DevOps](http://www.uml.org.cn/itil/201608101.asp)
[【第八期金融CIO论坛】DevOps时代合伙人张乐：大型互联网公司金融项目DevOps转型实践](http://www.ciotimes.com/txhhd/127012.html?utm_source=tuicool&utm_medium=referral)
[万达网络科技的DevOps平台架构解析](http://blog.itpub.net/31562043/viewspace-2284567/)
[加速企业敏捷的DevOps平台](http://blog.itpub.net/31562043/viewspace-2284567/)
[Periodic Table of DevOps Tools (v3)](https://xebialabs.com/periodic-table-of-devops-tools/)
[Devops时代，腾讯阿里的运维实践（附Devops58个开源工具） ](http://www.sohu.com/a/198439910_100038984)
[DevOps，研发运营一体化的时代已到来](http://baijiahao.baidu.com/s?id=1640289088037981327&wfr=spider&for=pc)

[Gitlab备份和恢复操作记录](https://www.cnblogs.com/kevingrace/p/7821529.html)
[实用运维之gitlab主从实时同步](https://www.jianshu.com/p/52de6a8d29d6)
[使用Mirroring同步主从Gitlab数据](http://mini.eastday.com/mobile/190630060851883.html)
[使用gitlab-mirrors从其它版本库同步代码](https://www.jianshu.com/p/54bd32c4862b)
[使用GitLab Mirrors同步Git仓库](https://www.jianshu.com/p/70b138f88514)
