---
title: MobaXterm基础用法与深度学习环境配置
tags:
  - MobaXterm
  - Conda
comments: true
mathjax: false
katex: false
highlight_shrink: false
aside: true
categories: 教程
description: 本文介绍通过MobaXterm配置深度学习环境和运行程序的基本过程。
copyright: true
date: 2025-05-13 17:05:00
updated: 2026-03-12 06:05:27
---


# 一、前言

在拥有 GPU 资源的课题组中，多人同时使用一台服务器是常见现象，这可以有效提高资源的利用率。而服务器一般运行的是 Linux 系统（以 Ubuntu 为代表），需要通过远程连接的方式使用。SSH 和 SFTP 是两个分别用于远程连接和文件传输的主流协议，为许多工具（如 X Shell、MobaXterm 等）所支持。本文介绍在 Windows 系统上通过 MobaXterm （功能强大，优先推荐）远程连接 Ubuntu 服务器，配置深度学习环境，以及运行程序的过程。

# 二、服务器连接

## 1. 软件安装

从[官网](https://mobaxterm.mobatek.net/download-home-edition.html)下载软件包，Portable Edition 解压后即可使用，Installer Edition 需要安装后才能使用，根据个人需要选择。

![MobaXterm 下载页面](https://media.rsune.top/blog/mobaxterm_dl.webp)

## 2. 软件使用

1. 确定自己需要连接服务器的 IP 地址和用户名，以及连接认证方式（密码或密钥），然后打开软件，按下图操作（如果使用密码则跳过第四步），新建连接会话并保存。

   ![MobaXterm 新建SSH连接](https://media.rsune.top/blog/mobaxterm_newConn.webp)

2. 打开连接会话

   下图是连接后软件的主界面，在终端输入命令后按回车键即可执行，操作结束后按 `Ctrl+D`键断开连接。

   ![MobaXterm 连接主界面](https://media.rsune.top/blog/mobaxterm_main.webp )

3. 会话配置复用和文件操作

   使用之前建立的会话配置进行连接，通过 SFTP 协议进行文件上传、下载等操作，见下图。

   ![MobaXterm SFTP使用](https://media.rsune.top/blog/mobaxterm_sftp.webp )

# 三、深度学习环境配置和使用

> 此处假定服务器上已经安装好了 Conda 环境（通过 Anaconda 或 Miniconda 安装），可以在终端执行 conda 命令。

## 1. 环境配置

```bash
# 使用Conda进行环境管理
conda env list # 查看当前系统的所有环境
# 新建环境，env_name为自定义的环境名，package_spec为待添加的包（可指定版本），方括号内的为可选内容，下同
conda create -n env_name [package_spec [package_spec]]	# 如 conda create -n python_3.7 python=3.7
conda env remove -n env_name	# 删除某个环境（删除前必须先退出该环境）
```

## 2. 环境使用

```bash
conda activate env_name		# 激活指定环境
conda info	# 查看当前环境的相关信息
conda list	# 查看当前环境已安装的包
conda install package_spec	# 安装指定包，如 conda install python=3.11（可只使用包名，不接版本号）
pip install package_spec	# 使用pip安装环境（当conda库不存在该包时再使用），如 pip install Django==1.7
# 删除指定环境中（默认为当前环境）的某些包，使用--all选项可删除该环境下的所有包
conda remove [-n env_name] [package_name [package_name]]	# 如 conda remove python	
pip uninstall package_name	# 使用pip删除环境中的指定包
python	/path/to/xxx.py		# 运行指定的python文件，如果在当前终端目录下则可直接使用文件名，无需加完整路径
python	# 进入交互式python环境，可输入代码（用分号结束）并执行，按 Ctrl + D 键退出，或使用exit()命令。
conda deactivate	# 退出环境
```

示例操作如下：

![MobaXterm 终端命令演示](https://media.rsune.top/blog/mobaxterm_demo_blur.webp )

## 3. 常用命令

```bash
nvidia-smi	# 查看显卡状况（温度、可用显存、使用率等）
free -h		# 查看内存状况（如可用内存容量）
ps -ef | grep python	# 查看所有python进程
kill pid	# 终止指定pid对应的进程
ping -c 3 baidu.com	# 测试网络状况
```

示例操作如下：

![MobaXterm 常用命令演示](https://media.rsune.top/blog/mobaxterm_demo2.webp )
