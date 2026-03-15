---
title: 通过LetsEncrypt申请免费SSL证书
comments: true
copyright_author: 饮冰
mathjax: false
katex: false
highlight_shrink: false
aside: true
abbrlink: 38af4d5f
date: 2023-06-03 22:56:29
tags: Linux
updated: 2026-03-07 19:45:00
categories: 教程
keywords:
description:
top_img:
cover:
toc:
toc_number:
toc_style_simple:
copyright: true
copyright_author_href:
copyright_url:
copyright_info:
aplayer:
---


# 一、前言

## 1、介绍

​	要在网站上启用HTTPS安全访问，需要从证书颁发机构（CA）获取证书。**Let's Encrypt**是一个提供免费证书的机构，只要能证明对域名具有所有权即可获取证书。证明域名所有权通常使用Web主机上运行的ACME协议客户端，而**Certbot**是该机构推荐使用的一款客户端工具。

​	**Let's Encrypt官网**：https://letsencrypt.org

​	**Certbot官网**：https://certbot.eff.org ，详细文档：https://eff-certbot.readthedocs.io/en/stable/

​	**Certbot官方教程**：https://certbot.eff.org/instructions ，在选择使用的Web服务器和运行的Linux系统后会出现相应的使用教程。本文操作环境为 Ubuntu Server 22.04 LTS 和 Nginx 1.18.0，请结合实际情况做出调整。

## 2、前提条件

* 对域名具有所有权
* 可以使用SSH访问域名对应的Web主机（具有命令行访问权限）
* Web主机可通过HTTP访问

# 二、安装证书

## 1、登录Web主机

​	使用SSH登录Web主机，登录用户需要具有sudo权限。

## 2、安装Certbot

​	官方推荐通过Snap（一种软件包格式）安装Certbot，本文也采用这种方式，若使用其他方法（如Docker和Pip），请参考 https://eff-certbot.readthedocs.io/en/stable/install.html 。

1. 安装Snapd

   Snapd是一个后台服务，用于管理Snap软件包。部分系统如Ubuntu 18.04及以上版本已预装Snapd，如需安装请参考 https://snapcraft.io/docs/installing-snapd 。

   ```bash
   # 如果不知道是否安装了snapd可以使用which命令查看
   which snap
   # 如果返回了一个路径，而不是 “snap not found”，则说明已安装
   ```

2. 移除先前安装的Certbot

   如果之前通过apt、yum等包管理器安装过Certbot，需要先将其卸载，以确保使用的是Snap安装的全新Certbot。

3. 安装Certbot

   snap应用的资源访问权限包括 strict 和 classic 两种，后者最宽松，详细介绍可参考：https://snapcraft.io/docs/explanation/security/snap-confinement/index.html

   ```bash
   sudo snap install --classic certbot	# --classic代表给予该级别的资源访问权限
   which certbot	# 检查certbot命令可否被调用，如果不能，使用命令sudo ln -s /snap/bin/certbot /usr/local/bin/certbot创建符号链接
   ```

## 3、使用Certbot

​	参照：https://eff-certbot.readthedocs.io/en/stable/using.html

   * 全自动运行

     获取证书，修改nginx配置文件，证书自动续期，一行命令即可完成。此方法会修改nginx配置文件以启用HTTPS，建议提前备份以备不时之需。

     ```bash
     sudo certbot --nginx	# --nginx指定用于获取和安装证书的插件
     # 运行此命令后会提示输入邮箱（可跳过），然后会让选择需要申请证书的域名（从nginx主机配置中读取，也可在命令行中通过-d选项指定），根据需要选择即可
     ```
     
   * 手动运行

     仅获取证书，不会修改nginx配置文件，需要自行配置。

     ```bash
     sudo certbot certonly --nginx
     sudo certbot certificates	# 打印证书信息（包括证书位置、关联的网站等）
     ```
     启用HTTPS后需要重载Nginx以监听443端口，同时检查Web服务器是否放行了443端口的入站请求。一个启用了HTTPS的Nginx网站配置示例如下：
     ```bash
     # /etc/nginx/sites-available/example.com
     server {
             server_name example.com;
             root /var/www/example.com;
             location / {
                     try_files $uri $uri/ =404;
             }
             listen [::]:443 ssl ipv6only=on;
             listen 443 ssl;
             ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; 
             ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; 
             include /etc/letsencrypt/options-ssl-nginx.conf; 
             ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
     }
     server {
             listen 80;
             listen [::]:80;
             server_name example.com;
             return 301 https://$host$request_uri;	# HTTP访问301重定向至HTTPS
     }
     ```


# 三、附录：概念理解及常用命令

## 1、证明域名所有权

​	规范中要求使用Challenge进行证明，有两种类型，**http** 和 **dns**，完成Challenge即可证明你对域名的所有权，进而申请证书。前者需要在Web主机的根目录下放置一个可以公网访问的文件，后者需要添加一条域名解析记录。

> 两种Challenge的详细说明参考网址：
>
> * http：https://datatracker.ietf.org/doc/html/rfc8555#section-8.3
>
> * dns：https://datatracker.ietf.org/doc/html/rfc8555#section-8.4

## 2、插件

​	证书的获取和安装一般通过插件实现，有些插件同时具有证书获取和安装的功能，有些却只具备一种功能。插件按来源可分为内置插件和第三方插件，其中后者由第三方开发者开发，大多数是针对某个具体的云服务/DNS解析提供商。插件可以单独使用，也可以组合使用（即一个用于获取证书，一个用于安装证书）。证书获取插件的功能是在完成Challenge之后将证书下载到Web主机中，而证书安装插件的功能是修改Web服务器的配置文件以应用证书（例如指定所获取证书的路径）。证书的安装可以手动完成，前提是要掌握Web服务器配置的方法。常用的Web服务器Apache和Nginx都有对应的插件，且能同时完成证书获取和安装的功能。插件的详细介绍参照：https://eff-certbot.readthedocs.io/en/stable/using.html#getting-certificates-and-choosing-plugins 。

## 3、证书类型

​	证书按照类型分为以下三种：

* 单域名证书：只针对一个域名生效，如适用于 `www.example.com` 的证书。
* 多域名证书：针对多个域名生效，如同时适用于`www.example.com` 和 `mail.example.com` 的证书。
* 泛域名证书：针对多个子域名同时生效，如适用于`*.example.com`的证书。

>注：
>
>1. 多域名证书和泛域名证书的区别在于，前者必须在申请证书时列出所有域名，而后者不需要。
>2. 泛域名证书，如适用于`*.examle.com`的证书不能用于`example.com`。
>3. 泛域名证书申请时一般采用的challenge类型为dns。

## 4、证书管理

​	获取到的证书通常保存在 /etc/letsencrypt/live/$domain 文件夹中，其中 $domian 代表证书的名字（一般是网站域名），该文件夹一般保存着 privkey.pem、fullchain.pem 两个文件（可能也会有 cert.pem 和 chain.pem ）。

> 证书管理一般包括以下内容：
> * 证书的创建：重新获取证书。
> * 证书的更新：证书具有一定有效期，需要在过期前更新证书以继续使用。
> * 证书的撤回：撤销（Revoke）一个证书，使其变为无效，一般在私钥泄露等情况下使用。
> * 证书的删除：删除已有的证书，在此之前要更新Web服务器的配置文件。

​	证书的更新可以由插件自动完成，也可以手动完成，其实质都是设置定时器，定期执行证书更新命令。

​	手动设置证书更新参见：https://eff-certbot.readthedocs.io/en/stable/using.html#setting-up-automated-renewal 。

## 5、常用命令
如命令执行后提示权限不足（ Permission Denied ），请使用sudo。
* snapd
    * snap help				  # 查看帮助信息
    * snap list	        		     # 列出已安装的所有包
    * snap find package	    	# 从远程仓库中查找包
    * snap install package		 # 安装包
    * snap remove packag                # 删除包
* certbot
  * certbot help                                           # 查看帮助信息
  * certbot certificates				# 查看所有证书
  * certbot delete --certname $name      # 删除指定的证书，其中$name是证书名
  * certbot show_account			  # 查看账户信息
  

