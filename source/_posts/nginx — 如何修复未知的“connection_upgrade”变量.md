---
title: nginx — 如何修复未知的“connection_upgrade”变量
date: 2024-11-28 15:22:59
categories:
 - nginx
tags: 
 - nginx
---

$connection_upgrade在使用 Websockets 或使用 nginx 配置生成器时，您可能会在 nginx 配置中遇到变量。
默认情况下，此$connection_upgrade变量不可用。但是，建议在反向代理设置中定义并使用它。
本教程将向您展示如何修复与连接升级相关的 nginx 未知变量消息！
问题：未知的“$connection_upgrade”变量
您可能会在（更新及之后）使用以下命令检查 nginx 配置时遇到此问题nginx -t：
> $ sudo nginx -t
> nginx: [emerg] unknown "connection_upgrade" variable  
> nginx: configuration file /etc/nginx/nginx.conf test failed
> 
该connection_upgrade变量不是全局 nginx 设置。然而，您会在整个互联网上的教程和代码片段中看到它。甚至nginx 公司也建议定义和使用connection_upgrade。让我们这样做吧！
close如果升级标头设置为 ，则此映射块告诉 nginx 正确设置相关的连接标头''。
将 map 块放入http你的 nginx 配置块中。nginx 配置的默认文件路径是/etc/nginx/nginx.conf。
下面是我们使用 map 块定义“变量”的 nginx 配置示例$connection_upgrade。
> /etc/nginx/nginx.conf
> user www-data;  
> worker_processes auto;  
> pid /run/nginx.pid;
> 
> events {  
>         multi_accept       on;
>         worker_connections 65535;
> }
> 
> http {  
>         sendfile on;
>         tcp_nopush on;
>         tcp_nodelay on;
>     ##
>         # Connection header for WebSocket reverse proxy
>         ##
>         map $http_upgrade $connection_upgrade {
>            default upgrade;
>          ''      close;
>      }
>       # further configurations …
> }

保存更新的 nginx 配置文件。然后，使用以下命令再次检查配置文件nginx -t：
```
$ sudo nginx -t

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
nginx: configuration file /etc/nginx/nginx.conf test is successful  
```
