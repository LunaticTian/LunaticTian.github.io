---
title: Docker 学习笔记
tags:
  - Docker
  - 服务端
abbrlink: 920e92d1
date: 2018-05-17 08:35:14
---

# My docker
学习Docker算是跟上潮流，但我这人不喜欢随大流。之所以动手去搭建是因为的确能给开发和多系统操作带来便利。<br>
虽然作为运维更需要更深层次的学习Docker，但是作为一个开发人员还是需要去了解Docker，知道它的工作方式原理是什么形式的，还有它的基本操作步骤。

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526527830744&di=7f482d07ab6f2d62dca0745405923f9a&imgtype=0&src=http%3A%2F%2Fs4.51cto.com%2Foss%2F201711%2F01%2Fa3b8e2c741aa9e6366595a7a87ffe3cb.jpg-wh_651x-s_2258130223.jpg)

<!--more-->
## What is Docker？

先来说说什么是Docker，Docker官方性的解释是：<br>
Docker是一个开源的引擎，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、bare metal、OpenStack 集群和其他的基础应用平台。

再来说说我的理解，Docker可以形象解释为一种隔离出来的虚拟机，但是他的确不是。每一个工作空间都是一个容器，一种口味的果汁。但是Docker给我们的不是一杯，而是可以根据我们的需求去制作出各式各样的果汁，以此下次我们换一个地方还需要在喝这么多果汁的时候只需要拿过来就行，而不必要在去制作。

## Effect

大致有三点，此处用阮一峰的说发：

（1）提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

（2）提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。

（3）组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

我在开头的时候说过，很多时候困扰开发人员是多开发环境的搭建，之前和同学开发的项目，在windows下跑的正常，然后放在服务器跑就出现环境原因，整整花了一天才找到问题的根源以及解决方法。

所以需要一个统一的环境成了一项很迫切需要解决的问题。

## OS 支持
很幸运的是Docker支持几乎全部平台：

* Mac
* Windows
* Ubuntu
* Debian
* CentOS
* Fedora
* 其他 Linux 发行版

## Docker Start
因为服务器跑的是CentOS但是以防出现问题，手上拿的是虚拟机跑的CentOS7。

linux要特别注意的是内核版本必须在3.10以上，所以在安装之前要知道系统内核是否在要求及要求以上。

查看系统内核版本号及系统名称
	
	uname -a

阿里云CentOS6：
![](http://img.lunatic.wang/%E9%98%BF%E9%87%8C%E4%BA%91Uname.PNG)

虚拟机上的CentOS7：
![](http://img.lunatic.wang/%E6%9C%AC%E6%9C%BAUname.PNG)

当然阿里云的内核明显达不到要求，可以更换CentOS7以上，但是我们跑的项目以及一些在服务器上设置好的难免需要重新配置，所以可以直接在CentOS6里升级内核以达到Docker要求。

这里我选择vm上的CentOS7来做相关操作。

## Installation



CentOS集成yum，所以很方便的使用yum来安装Docker。

先来看看系统是否有yum：

我的方法是直接输入yum看系统状态，如果有帮助信息可知安装信息

	# yum

安装yum的提示：

![](http://img.lunatic.wang/yum.PNG)

<br>


	yum install docker //yum安装docker


在安装过程中会需要询问输入几次Y/N，直接Y就可。

![](http://img.lunatic.wang/%E5%AE%89%E8%A3%85docker.PNG)

提示Complete！安装完成！

![](http://img.lunatic.wang/docker%E5%AE%89%E8%A3%85%E5%AE%8C%E6%88%90.PNG)

确认docker是否安装成功：

	# docker

![](http://img.lunatic.wang/%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9F%E6%A3%80%E6%9F%A5.PNG)


OK,安装大功告成！

## Start Docker

安装完成之后，我们还需要启动Docker：

	# sudo systemctl start docker
	//启动docker
	# sudo systemctl enable docker 
	//开机启动


## Choice

安装完docker之后我们就可以选择镜像了。


此镜像与我们平常所见镜像从使用过程中所见相同，由镜像所生成的容器也就相当于一个隔离的系统。

每一个容器里都包含我们所需要的某个环境。

至此，我以python环境为例：

首先我们先看linux上的Docker有哪些环境

	# docker images

![](http://img.lunatic.wang/docker%E6%9C%89%E5%93%AA%E4%BA%9B%E7%8E%AF%E5%A2%83.PNG)

安装是不带有任何环境的，所以需要我们去下载

	# docker search python

![](http://img.lunatic.wang/docker%E6%9F%A5%E6%89%BEpython.PNG)
这里有很有python相关的镜像，但是python有python2与python3，很多时候CentOS是内置有python2，我在开发的时候是基于python3的，所以这里我下载python3的环境

	# docker pull python:3
	//docker pull <镜像名称>:<版本> lastest为最新版

## Create python

pull python之后，我们就可以创建python3的容器：

	# docker run -i -t --name python3 python /bin/bash
	// -i -t 前台显示运行状态的输入输出
	// --name pyhton3   // 容器名称为python3
	//run命令执行了两步操作即将镜像放入容器中，并且运行容器。

![](http://img.lunatic.wang/%E5%88%9B%E5%BB%BApython3%E5%AE%B9%E5%99%A8.PNG)

当出来新的71f31...说明以及进入容器了，当然我们也可以在外部操作。


测试python环境：

	# python -version

如有错误也可使用：
	# python

![](http://img.lunatic.wang/python%E7%89%88%E6%9C%AC.PNG)


## Run Python

OK一切正常，接下来我们再来测试一些脚本。
之前写了一个爬虫的程序，需要用到beautifulsoup4，所以这里安装beautifulsoup4。

	# pip install beautifulsoup4
	# pip install requests
	# pip install lxml

我们将脚本cp到容器内：

我把脚本存放在/root/python中，然后我将它复制到容器内部进行运行。

首先退出容器，回到CentOS。

	# exit
	//复制脚本
	# docker cp /root/python/3dmGame.py python3:/root


这里exit已经将容器stop了，我们重启启动容器才能进行连接。不然会报错。
	
	//启动python3容器
	# docker start python3
	//连接python3容器
	# docker attach python3
	//进入容器root目录
	# cd /root
	//查看文件
	# ls
	
![](http://img.lunatic.wang/cp%E8%84%9A%E6%9C%AC.PNG)

有我需要的3dmGame.py的脚本，我们看能不能运行。

	# python 3dmGame.py


![](http://img.lunatic.wang/123123123.jpg)

成功爬取~


该脚本的github：[Python-Reptilian](https://github.com/LunaticTian/Python-Reptilian)

## Tomcat & Tips 

其实我们在跑容器的时候大多数都是后台运行，譬如tomcat。
我们并不需要进入启动什么，只要它在跑就行了。

tomcat：


	# docker run --name tomcat1 -p 8080:8080 -v$PWD/test:/usr/local/tomcat/webapps/test -d tomcat 
	//-p 8080:8080 前8080为宿主机器的端口，后为容器端口，这里将宿主的8080映射到了容器的8080
	//-v$PWD/test:/usr/local/tomcat/webapps/test 将test挂在到容器的webapps/test目录下。
	//-d 后台运行

我们在发布的时候，只需要使用 docker cp 将文件复制到容器的webapps上即可：

	# docker cp test/Sdnote.war tomcat1:/usr/local/tomcat/webapps



外部访问项目的结果：

![](http://img.lunatic.wang/TIM%E6%88%AA%E5%9B%BE201805171520402.jpg)


不管是后台还是前台运行，容器总是在跑的，我们可以通过ps来查看当前运行的容器：

	# docker ps

我们也可以通过exec来连接容器或者从外部运行容器：

上面的连接python3可以变成：

	docker exec -it python3 /bin/bash
	// -it 前台输出输入
	// /bin/bash shell命令行

也可以直接从外部执行容器：

	docker exec python echo "hello world"


![](http://img.lunatic.wang/1231231231.jpg)



## Bey python3?

假设python3容器我们不需要了，要删除它...

Well....OK....

查看docker是否运行了python3环境
	
	# docker ps -a
	
如果正在运行：

	# docker stop python3

删除！

	# docker rm python3

或者我们删除镜像

	# docker rmi python：latest
	//lastet 标签，最新的版本
	或者
	# docker rmi python:3

![](http://img.lunatic.wang/python%20%E5%88%A0%E9%99%A4.jpg)


## No End

至此，Docker的入门算是OK了，后面我会继续学习Docker的进阶。




Emmmm，docker还是挺有意思的....

