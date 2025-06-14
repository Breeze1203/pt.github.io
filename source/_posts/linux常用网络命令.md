---
title: "Linux常用网络命令"
date: 2025-06-11 19:23:37
categories: linux
tags: linux
---

### Linux常用网络命令

1\. ping：用于测试网络连接的连通性。可以通过发送ICMP Echo请求到指定IP地址来检查是否能够收到响应。

2\. ifconfig：用于配置和显示网络接口的信息，包括IP地址、子网掩码、MAC地址等。也可以使用ifconfig来启用或禁用网络接口。

3\. netstat：用于显示网络连接信息、路由表、网络接口统计信息等。常用的选项有-a（显示所有连接）、-n（以数字形式显示地址和端口）、-r（显示路由表）。

4\. nslookup：用于进行域名解析。可以用来查询特定域名的IP地址或反向查询IP地址对应的域名。

5\. traceroute：用于追踪数据包从源地址到目的地址的路径。通过显示经过的每个路由器的IP地址以及响应时间，可以帮助定位网络延迟或故障点。

6\. dig：用于域名查询和信息收集。可以查找特定域名的DNS记录、查询DNS服务器、获取域名的详细信息等。

7\. wget：用于从网络上下载文件。可以通过URL下载文件，并支持断点续传、限速、代理等功能。

8\. curl：功能类似于wget，可以用于从网络上获取文件。它还支持更多的协议和操作选项，可以发送不同类型的请求（GET、POST等）。

9\. ssh：用于远程登录到其他计算机。通过安全的加密协议，可以远程管理和操作其他计算机。

10\. ftp：用于在本地和远程计算机之间传输文件。可以使用ftp命令登录到远程服务器，上传和下载文件，以及管理文件和目录。
