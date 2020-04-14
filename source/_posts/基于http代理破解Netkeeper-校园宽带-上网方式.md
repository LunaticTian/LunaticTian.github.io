---
title: 基于http代理破解Netkeeper(校园宽带)上网方式
tags:
  - 杂碎
  - 鼓捣
abbrlink: 8d93fad
date: 2018-10-06 20:40:27
---
## 灯泡
室友把刚闲置的主机拿到我这里用来开黑，等于多了一台设备，但是学校是垄断宽带运营商，只有一台PC能够联网，所以打算把主机拿来日常使用
笔记本用来做开发。用惯了pc拓展显示器之后，在用独立的双PC来使用还是挺别扭的，特别是笔记本还不能联网。

Netkeeper（校园网）的限制机制是限制无线网共享方式，监测系统的热点以及网卡和热点程序。网络上有很多破解方式，曾经用过CrazyChen[的Simple Netkeeper](https://simplenetkeeper.com/index.html)(停止维护)，以及[Simplenktools](http://www.simplenktools.cn/)(破解网络共享)，后者由于网卡原因wifi经常掉线，索性就放弃了。


本来在网络共享上就不抱有期望了，但想能够简化双机的操作方式，所以思考能否用windows自带的远程连接来连接桌面，如果能够连接成功可以方便许多。

<!--more-->


### 网络连接方式


- 接入寝室的网络接口连入了一台路由器(没有插入wan口)
- 路由器又连接了一台分线器
- 分线器连接了主机以及笔记本

![](http://cdn.lunatic.wang/tuoputu.png)


### 首先我查看笔记本ip地址

> CMD 

> $ ipconfig /all

![](http://cdn.lunatic.wang/bijibencmd.jpg)

两个ip，分别是100.64.98.103以及192.168.137.1。后者是路由器分配的ip，前者100的ip是内网分配的ip，联想到学校的FTP服务器在寝室也可以上传下载，这个ip是学校内网分配的ip。说明主机也应该分配了一个相同网关的ip。

### 查看主机ip

![](http://cdn.lunatic.wang/zhujiIP.PNG)

主机分配100网关ip以及由Netkeeper分配的27的公网ip，主机也在100的网关上，所以说明两者在同一内网中。

### ping

相互ping网，能否ping通，证明远程连接可行。

#### 主机ping笔记本

![](http://cdn.lunatic.wang/zhujiping.PNG)

**成功**

#### 笔记本ping主机

![](http://cdn.lunatic.wang/bijibenping.jpg)

并不能ping通，但是主机笔记本却正常说明还是防火墙问题，开启防火墙的高级入站规则：

![](http://cdn.lunatic.wang/zhujifanghuoqiang.PNG)

文件和打印机共享(回显请求)是没有启用的，启用后：

![](http://cdn.lunatic.wang/xiugaifanghuoqiang.PNG)

再次用笔记本ping主机：

![](http://cdn.lunatic.wang/xiugaibijibenping.jpg)

在解决了内网连通问题后在进行远程连接。


### 远程连接

**被远程的Windows版本要在专业版以上**

远程连接笔记本

> cmd  
> $ mstsc

![](http://cdn.lunatic.wang/yuanchen1.PNG)

![](http://cdn.lunatic.wang/yuanchen2.PNG)

**成功**

可以远程访问桌面之后，很多开发都能很方便的完成，而不需要操作两套设备。唯一可惜的是笔记本没有网，这很难受。

### 为什么是灯泡

![](http://cdn.lunatic.wang/u=1422934676,129906231&fm=26&gp=0.jpg)

完成以上，突然想到：

- 处于相同内网
- 可以ping通
- 能够远程连接

满足这些条件，既然处于相同内网，说明内部网络可以共享或者**代理**某些程序或者**网络**。

在不联网的情况下，笔记本和主机能够相互代理，访问对方的网络和网络环境。那么，如果主机或者笔记本有网，是不是也可以通过代理来共享网络呢？

那我就试试。


## 三选一的方式

既然说到代理，目前流行三种：

- Shadowsocks
- VPN
- HTTP代理



### Shadowsocks
首先我想到的是架设shadowsocks服务器来共享网络，但是shadowsocks主要用于网页访问。虽然有此缺点，但是我还是安装了python环境以便安装服务端。

[Windows下三分钟搭建Shadowoscks服务器端](https://www.librehat.com/three-minutes-to-set-up-shadowsocks-server-on-windows/)


主机架设好之后，笔记本通过Shadowoscks甚至都不能ping通主机的Shadowoscks服务器，在查阅了很多解决方案后发现都不能成功，所以这个方法就被放弃掉了。但是显然失败原因很有可能是主机的环境原因，所以客户端无法访问。


### VPN

思考之后显然vpn的代理方式最好，第一vpn的架设在于内网，第二vpn不用过多配置，所有的流量全部走vpn，也就是全局。

之前在用windows server 成功配置了vpn，所以更倾向于这种方案。在主机（win10）中正确配置了vpn服务器之后，笔记本出现了Shadowsocks相同的问题，就是无法连接到vpn网络，在验证用户后，一直无法连接。并且很多人都说在win10中配置的vpn无法访问，甚至主机自己都不能连接本机的vpn，说起来还是挺好笑的。很多方法都不能解决问题，这导致心里的期望越来越低。在这上面花费了很长时间仍然不能解决问题，暂时性的放弃了。并不是这个方案不行，可以说，若是windows server，应该可以配置好，但是windows server 显然不够日常使用，并且我也不想换系统，所以暂时放弃这个方案。


### HTTP代理

这是最后的方法，也许有点命中注定的意思。

windows本身不提供http的搭建方案，所有需要第三方的软件或者方案。

所以就先这样。


## 再见曙光

在寻找了很多方案之后，找到了这款软件[CCProxy](http://www.ccproxy.com/),它可以很方便的搭建一个http的代理，并且体积很小，搭建方便，适合后台运行。


这里CCProxy提供免费版，虽然只提供三个用户(ip)访问，但是对于个人用户完全足够。
如果有更多用户需求，可以购买(虽然很贵)，也可以自寻注册码。

### 配置CCProxy

点击设置，下方有IP选择。这里显示了两个ip,由于是内网，所以这里选择100网关的ip。

![](http://cdn.lunatic.wang/xuanzeip.png)

然后在选择点击所需要的代理，这里除开socks其他都点了，其实没啥必要。socks没选择是因为本身主机有Shadowsocks代理占用1080端口，若不修改端口则Shadowsocks会被报端口占用的错误。

![](http://cdn.lunatic.wang/duankouxuanze.png)

账号有账号管理功能，由于内网暂时没有保密必要，所以这里就没有限制账户以及ip限制。

至此基础的配置已经完成。

### 激动人心

在笔记本中选择Internet属性，局域网设置，代理服务器填写了配置CCProxy的ip以及808的端口。

为防止某些测试会回调127.0.0.1的本地ip,所以我勾选了对于本地..
![](http://cdn.lunatic.wang/ceshi.png)

完成设置以后访问了baidu.com。

![](http://cdn.lunatic.wang/baidu.PNG)

完成基于http代理的共享方式，此刻内心还是挺激动的。

基于此，所有设备在接入内网后，只要在同一网段，并且能够ping通则可以用此代理来共享校园网。


## Proxifier


虽然成功完成了http的设置，但是此方式不像vpn能够全局访问，仅仅只能浏览网站，所以这里浏览网站是不够的，这里需要进行全局的HTTP代理。


### PC

这里需要用到之前配合Shadowsocks进行全局的**Proxifier**。

下载**[Proxifier](http://www.proxifier.com/)**,安装。

打开Proxifier，点击代理服务器,点击编辑填写ip以及端口。

可以选择点击检查，测试，没问题即可点击确定。

![](http://cdn.lunatic.wang/peizhiProxifier.png)


接着点击代理规则，在目标主机中加入被代理的ip。

![](http://cdn.lunatic.wang/bianxieguize.png)

至此代理完成，关闭Internet属性中局域网勾选。

登陆TIM测试。

![](http://cdn.lunatic.wang/qq.png)

成功！

### 安卓

安卓需要下载ProxyDroid进行搭配。

由于我的手机是8.0，root测试发现很多问题，所以就没有成功，不过8.0之前版本可以进行测试。

[教你在Android手机上使用全局代理！](https://blog.csdn.net/testcs_dn/article/details/78526468)

## Thoughts

虽然作为大四狗终于完成了心中关于校园网的心病，但是还是遗憾没有过早的想到这样一个解决方案。基于http代理形式的网络共享优点的确很多，但是全局配置困难仍然是一个很麻烦的点。特别是移动端(ios,Android)的配置更是异常的复杂，所以最好的解决方案还是用基于VPN搭建方式，这样内网的网络稳定延迟低，搭配全局完全可以整个个人设备的使用。

我想总该为大学后来者，为同仁献出自己的绵薄之力。

但是，这还不够。








