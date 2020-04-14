---
title: Flyve-MDM环境搭建
tags:
  - 服务端
  - Linux
abbrlink: 9e4406e1
date: 2019-08-14 11:27:03
---
# Flyve MDM 环境搭建

## 前言
很久没有写博客了，这半年一直在忙找工作和忙工作，很少能有时间静下心来写。Flyve-MDM官方只有视频教程和docker搭建，没有具体的文字搭建文档，这里记录我搭建Flyve-MDM的搭建过程。


<!--more-->
## 系统环境搭建
- sudo apt-get update

### Apache2 

- sudo apt install apache2
- sudo systemctl status apache2

### PHP
- sudo apt-get install software-properties-common python-software-properties 
- sudo add-apt-repository ppa:ondrej/php && sudo apt-get update
- sudo apt-get -y install php7.2

### 配置PHP Apache2
- sudo apt-get install libapache2-mod-php

### MySql 
- sudo apt-get install mysql-server
- sudo mysql_secure_installation/password:root



## GLPI 搭建

### 获取GLPI
切换成root用户，进入/var/www/html

- sudo su
- cd /var/www/html

进入[glpi-project/9.3.3](https://github.com/glpi-project/glpi/releases/tag/9.3.3)，下载项目文件。同时解压到html目录中。

- wget [https://github.com/glpi-project/glpi/releases/download/9.3.3/glpi-9.3.3.tgz]()
- tar -xvzf glpi-9.3.3.tgz
- rm -rf glpi-9.3.3.tgz

进入plugins，下载[fusioninventory-for-glpi/9.3+1.3](https://github.com/fusioninventory/fusioninventory-for-glpi/releases),解压。

- cd glpi 
- wget [https://github.com/fusioninventory/fusioninventory-for-glpi/releases/download/glpi9.3%2B1.3/fusioninventory-9.3+1.3.tar.bz2]()
- tar -xvjf fusioninventory-9.3+1.3.tar.bz2
- rm -rf fusioninventory-9.3+1.3.tar.bz2

进入[flyve-mdm](https://github.com/flyve-mdm/glpi-plugin/releases),下载该项目2.00版本,进入flyvemdm同时切换成非root用户。

- wget [https://github.com/flyve-mdm/glpi-plugin/releases/download/v2.0.0/glpi-flyvemdm-2.0.0.tar.bz2]()
- tar -xvjf glpi-flyvemdm-2.0.0.tar.bz2
- cd flyvemdm
- su @name
- sudo apt install composer
- composer install --no-dev

## 配置GLPI

### 初始化配置
进入glpi项目根目录，查看本机地址或使用localhost,进入项目网页，第一次进入会自动跳转到项目环境的配置界面。

- cd /var/www/html/glpi
- 使用浏览器输入 http://localhost/glpi

进入到Step 0提示很多PHP依赖尚未安装，安装PHP依赖,重启apache2

- sudo apt-get install php 7.2-ldap
- sudo apt-get install php 7.2-imap
- sudo apt-get install php 7.2-apcu
- sudo apt-get install php 7.2-xmlrpc
- sudo apt-get install php 7.2-cas
- sudo apt-get install php 7.2-mbstring
- sudo apt-get install php 7.2-gd
- chmod -R 777 files
- chmod -R 777 config
- service apache2 restart 

### 配置MySql
进入mysql，添加用户glpiuser

- mysql -u root -p 
	- CREATE USER 'glpiuser'@'localhost' IDENTIFIED BY 'admin'
	- GRANT ALL PRIVILEGES ON *.* TO 'glpiuser'@'localhost' WITH GRANT OPTION;

再次进入[http://localhost/glpi](http://localhost/glpi),跳转到install页面，在登录页面输入相关信息,内置的脚本文件会创建好相应的数据库文件。

- SQL server：localhost
- SQL user：glpiuser
- SQL password：admin
- Create a new database or use an existing one:glpi_ad



### 配置GLPI网页信息

- 根据首页提示修改默认多个账号密码
- 删除install.php文件
	- cd /var/www/html/glpi/install
	- rm -rf install.php
- 点击Setup/Plugins
	- 点击FusionInventory/install
	- 安装完毕点击Enable
- 安装php-zip
	- sudo apt-get install php7.2-zip
- 重启apach2
- 进入Setup/Plugins
	- 点击zip/install/安装完毕点击Enable

![](http://cdn.lunatic.wang/glpi1.png)

- 点击进入Setup/Notifications
	- yes
	- yes
	- yes
		- 点击 Email followyps configuration
		- Way of sending emails/SMTP+TLS
			- 按照提示填空
			- check certificate/yes
			- 此项是flyve mdm系统邮件配置需开启TLS
![](http://cdn.lunatic.wang/glpi2.png)
![](http://cdn.lunatic.wang/glpi3.png)

## 配置 Dashboard 

- cd /var/www/html
- wget https://github.com/LunaticTian/glpi-plugin/releases/download/dashboard/dashboard.zip
- unzip -o dashboard.zip
- rm -rf dashboard.zip
- cd dashboard
- rm -rf config.example.js
- nano config.js

<center>![](http://cdn.lunatic.wang/glpi4.png)</center>

进入[http://127.0.0.1/glpi](http://127.0.0.1/glpi),点击Setup/General/API，复制URL of the API，Enable Rest API/yes，Enable login with credentials/yes，Enable login with external token/yes。

- 填入glpiApiLink中
- 填入bugsnag中



## 配置完成