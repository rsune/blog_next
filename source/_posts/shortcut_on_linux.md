---
title: Linux下应用快捷调用
tags: Linux
categories: 教程
comments: true
copyright_author: 饮冰
mathjax: false
katex: false
highlight_shrink: false
aside: true
abbrlink: 911d0c70
date: 2023-04-04 20:20:31
updated: 2026-01-23 20:20:00
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


**说明**：本文介绍在Linux下创建应用快捷方式的几种方法，下文用**程序**指代Linux下的**可执行文件**。

## 1、导出环境变量

> * 适用：Shell
> * 优点：操作后对文件夹下的所有程序均生效
> * 缺点：只能按照可执行文件的名称调用
> * 使用：在Shell中通过程序名调用程序

​	原理：将程序加入Shell命令搜索范围

根据需要修改环境配置文件，将应用程序位置写入PATH环境变量中。

```bash
#假设/usr/local/app/bin文件夹内放着两个可执行文件start和shutdown
#修改~/.bashrc，在最后写入以下内容
APP_HOME=/usr/local/app
PATH=.:$PATH:$APP_HOME/bin
export $PATH

#修改完成后使用source命令使修改生效，在Shell中通过程序名即可调用程序
user@localhost:~$start
```

## 2、创建符号链接

> * 适用：Shell
> * 优点：可以自定义链接文件名；不需要修改环境变量
> * 缺点：需要针对每个程序分别设置
> * 使用：在Shell中通过链接文件名使用

​	原理：在PATH环境变量指定的路径内放入指向程序的符号链接文件

```bash
#假定~/bin/very_long_app_name是一个程序
#假定/usr/bin已在PATH环境变量中
#在Shell中使用如下命令在/usr/bin内创建指向程序的符号链接文件app
sudo ln -s ~/bin/very_long_app_name /usr/bin/app

#命令执行完成后在Shell中通过符号链接名即可调用程序
user@localhost:~$app
```

## 3、创建快捷方式

> * 适用：桌面
> * 优点：使用方便
> * 缺点：只适用于桌面版系统
> * 使用：双击快捷方式

​	原理：在桌面文件夹内放置一个类似于Windows中应用快捷方式的.desktop文件

>假定/usr/local/bin/app是一个程序，/usr/local/res/app.ico是一个图标文件，在桌面文件夹内创建一个app.desktop文件，内容如下：
>
>[Desktop Entry]
>
>Type=Application
>
>Name=App
>
>Exec=/usr/local/bin/app
>
>Icon=/usr/local/res/app.ico
>
>Terminal=false

**注**：核心配置项包括Type、Name、Exec和Terminal；不同系统的桌面文件夹可能不同，Deepin和Ubuntu系统中桌面文件夹为~/Desktop

部分应用在安装时会自动创建桌面快捷方式，如果想自己添加桌面快捷方式的话可以采用这种方法。

参考：https://specifications.freedesktop.org/desktop-entry-spec/latest/
