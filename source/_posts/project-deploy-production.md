---
title: 借助宝塔面板部署项目（vue+node+mysql）
date: 2020-04-07 16:52:57
tags:
	- 部署
categories: 	
    - 部署
cover: http://spider.ws.126.net/d6020de6eac38409cb32f89daf8a7a52.jpeg
---

服务器使用的是阿里云CentOS Linux release 8.1.1911。前端使用的是vue+vant-ui，后端是koa2，数据库使用的是mysql。

## 配置宝塔面板

 https://www.bt.cn/ 

**Centos安装命令：**

```
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

云服务器中输入该代码，安装成功后会在命令行工具的最下面几行看到一个网址还有用户名密码这些信息。

{%asset_img p1.png%}

 **注意：有一点非常重要，这里生成的网址的端口号为8888，但是如果你直接访问的话是会被拒绝的，因为阿里云上面我们还需要做一些配置** 

{%asset_img p2.png%}

{%asset_img p3.png%}

{%asset_img p4.png%}

配置安全组规则端口号为8888，成功后即可进入宝塔面板登录界面。

登录成功后进入软件商店下载所需的软件：nginx、mysql、PM2管理器（ node.js管理器，内置 node.js + npm + nvm + pm2），数据库如果使用mongodb则可下载mongodb。

{%asset_img p5.png%}

## 添加数据库并使用navicat连接上服务器数据库

{%asset_img p11.png%}

由于mysql端口为3306，则需要在阿里云添加安全组规则

{%asset_img p6.png%}

但是本人配置了之后还是不行，navicat一直连接不上服务器(报错error)，之后花了很长时间才发现3306端口在防火墙也需要放行。

{%asset_img p7.png%}

点击放行之后即可连接。

## 设置PM2管理

宝塔面板中打开先前下载成功的pm2管理器

{%asset_img p8.png%}

点击添加，成功后启动即可通过服务器ip地址加端口加上端口号进行访问。如果需使用域名，点击映射，填写上你的域名即可。

## nginx反向代理

打开之前下载成功的nginx，修改配置。listen为代理端口号，server_name为你的域名或服务器IP地址，root为项目根路径，index就不多说了吧。proxy_pass是将服务器本地3333端口的项目代理到80端口，即访问该项目不需要添加端口号了。

{%asset_img p9.png%}

成功之后项目可以访问的到了，但是静态资源访问不了。

于是乎

{%asset_img p10.png%}

在下面配置静态资源的地方加上你的root为public即可。

项目访问地址 http://39.101.198.23