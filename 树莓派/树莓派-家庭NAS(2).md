树莓派-家庭NAS(1)   https://www.jianshu.com/p/9be7ada37863
树莓派-家庭NAS(2)   https://www.jianshu.com/p/91405ca824b8
树莓派-家庭NAS(3)   https://www.jianshu.com/p/80777ed85246

## 内网穿透选型
上一篇文章中介绍了，家用NAS树莓派的整体方案。在整体解决方案中说明的搭建整体方案的第一步是解决访问问题。本文将主要介绍访问解决方法。

内部网络穿透技术可以分为NAT、DDNS、反向代理和VPN。这里就不介绍这些方式，可以查看参考中的内容进行了解。这里说明他们的大概工作原理用于选型工作。
*   #### NAT技术：
    内网中的机器怎样访问公网上的网站？内网机器发送的数据可以经过Router转发的公网上，那公网上返回的数据怎么到达内网的机器内，这个过程就是NAT技术。主要技术是通过动态端口映射技术完成。
![NAT技术](https://upload-images.jianshu.io/upload_images/2454595-97bafd343b410f49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   #### DDNS技术：

    DDNS即动态域名解析，是将用户的动态IP地址映射到一个固定的域名解析服务上，用户每次连接网络的时候，客户端程序就会通过信息传递把该主机的动态IP地址传送给位于服务商主机上的服务器程序，服务程序负责提供DNS服务并实现动态域名解析。就是说DDNS捕获用户每次变化的IP地址，然后将其与域名相对应，这样域名就可以始终解析到非固定IP的服务器上，互联网用户通过本地的域名服务器获得网站域名的IP地址，从而可以访问网站的服务。
![DDNS技术](https://upload-images.jianshu.io/upload_images/2454595-1b74500af260c70d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   #### 反向代理：

    反向代理就是经常说的，有一台公网服务器和私网服务器在一个网络内。公网服务器上构建一个服务，把用户请求转发到死亡服务器上就可以了。
![反向代理技术](https://upload-images.jianshu.io/upload_images/2454595-2903a9121d8d7b86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

*   #### VPN技术：

    VPN技术就是将两个私网通过隧道技术组合成一个子网。
![VPN技术](https://upload-images.jianshu.io/upload_images/2454595-61325c7c60250848.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 方案选择
本次我们的方向是家用NAS，所以，网络环境也是家庭环境。所以，可能需要组合各种内网穿越技术才可以满足整体要求。所以，现在开始组织方案。

![家用NAS](https://upload-images.jianshu.io/upload_images/2454595-2119327a97c04d34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在本方案中家用网络使用的是电信网，电信网是有公网IP的。在没有公网IP的通信服务商，请自行选择其他方案。然后，再使用NAT技术的端口静态映射，将树莓派上的服务发布出去。使用DDNS技术的域名动态解析将路由器域名发布到域名上。

## 实现
0. 先把树莓派的内网IP设置为静态IP。
1. 实现上图中的第七步，在路由器上设置端口映射。因为自家使用的路由器不一样，我就不截图了。几乎所有的家用路由器都是支持端口映射的。
2. 在树莓派上编写代码，实现上图中1,2,3步。我使用的是NETGEAR R6200V2，所以根据路由器的特点进行了公网IP的获取工作。
3. 配置定时执行过程。

定时执行（cron）代码：
```
*/2 * * * * root /home/pi/update_public_ip.py > /dev/null 2>&1 &
```
更新域名IP代码（python）：
```
#!/usr/bin/python2.7
#-*-coding:utf-8-*-

import os
import sys
import httplib
import urllib2
import urllib
import base64
import cookielib

# 获取网管地址，即路由器地址
def getGateway():
  return "172.25.1.1"

# 获取路由器上的公网IP。因为实在路由器上拨号上网的，所以从路由器上获取公网IP
def getPublicIP(ip, user, password):
  base64string = base64.b64encode('%s:%s' % (user, password))
  headers = {"Host": ip,
             "User-Agent": "Mozilla/5.0 (Windows NT 5.1; rv:52.0) Gecko/20100101 Firefox/52.0",
             "Accept": "*/*",
             "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3",
             "Accept-Encoding": "gzip, deflate",
             "Referer": "https://172.25.1.1/RST_st_poe.htm",
             "Connection": "keep-alive",
             "Authorization":"Basic %s"  % base64string}
  # 登录服务器，获取Cookie
  conn = httplib.HTTPConnection(ip, 80)
  conn.request("GET", "", None, headers)
  response = conn.getresponse()
  cookie = response.getheader("set-cookie")
  headers["Cookie"] = cookie
  conn.close()

  # 获取公网IP地址所在页面
  conn = httplib.HTTPConnection(ip, 80)
  conn.request("GET", "RST_st_poe.htm", None, headers)
  response = conn.getresponse()
  result = response.read()

  # 解析页面中内容，分解出IP地址
  start_index = result.find("IP地址</B></td>")
  start_index = result.find("<TD NOWRAP>", start_index)
  end_index = result.find("</td>", start_index)
  result = result[start_index:end_index]
  result = result[len("<TD NOWRAP>"):]
  conn.close()

  # 登出路由器
  conn = httplib.HTTPConnection(ip, 80)
  conn.request("GET", "LGO_logout.htm", None, headers)
  response = conn.getresponse()
  conn.close()

  # 返回IP地址
  return result

# 更新二级域名的IP
def updateDomain(ip):
  os.system("curl \"http://update.dnsexit.com/RemoteUpdate.sv?login=XXXXXX&password=XXXXXX&host=XX.XXX.X&myip=%s\"" % ip)

# 更新二级域名的主流程
if __name__ == "__main__":
  public_ip = getPublicIP(getGateway(), "XXXXX", "XXXXX")
  print "public ip : %s" % public_ip
  updateDomain(public_ip)
```

## 参考：
[一分钟实现内网穿透（ngrok服务器搭建）](https://blog.csdn.net/zhangguo5/article/details/77848658)
