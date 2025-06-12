---
title: linux启动java服务
date: 2024-11-18 13:59:43
categories: JAVA
tags: 
 - JAVA
---

###### linux启动java程序
1. shell脚本启动
```
#!/bin/bash

# 使用which命令获取java的路径
JAVA_PATH=$(which java)

# 检查是否成功获取到了java的路径
if [ -z "$JAVA_PATH" ]; then
  echo "错误：未找到java可执行文件。"
  exit 1
fi

# 提取JAVA_HOME路径
JAVA_HOME=$(dirname "$JAVA_PATH")/..
if [ ! -d "$JAVA_HOME" ]; then
  echo "错误：无法从java路径确定JAVA_HOME。"
  exit 1
fi

# 提示用户输入jar包的路径
echo "请输入jar包的完整路径："
read JARFILE

# 检查是否输入了jar包路径
if [ -z "$JARFILE" ]; then
  echo "错误：未输入jar包路径。"
  exit 1
fi

# 检查jar文件是否存在
if [ ! -f "$JARFILE" ]; then
  echo "错误：指定的jar文件不存在。"
  exit 1
fi

# 提示用户输入执行的用户
echo "请输入用于运行应用程序的用户："
read RUN_AS_USER

# 检查是否输入了用户
if [ -z "$RUN_AS_USER" ]; then
  echo "错误：未输入执行的用户。"
  exit 1
fi

# 检查用户是否存在
if ! id "$RUN_AS_USER" &>/dev/null; then
  echo "错误：用户 '$RUN_AS_USER' 不存在。"
  exit 1
fi

# 设置其他环境变量
MODE=service
JAVA_OPTS="-Xms512m -Xmx1024m"

# 使用指定的用户启动JAR文件，并将其放到后台执行
sudo -u "$RUN_AS_USER" nohup "$JAVA_HOME/bin/java" $JAVA_OPTS -jar "$JARFILE" > /dev/null 2>&1 &
```
2. 当做service启动
```
--------------------------

/etc/systemd/system
--------------------------

vim java.service

--------------------------
[Unit]
Description=myapp
After=syslog.target

[Service]
User=root
ExecStart=/usr/java/bin/java -jar /root/work/code/admin-0.0.1-SNAPSHOT.jar
StandardOutput=append:/root/work/jar.log
StandardError=append:/root/work/jar.log
SuccessExitStatus=143
Restart=on-failure

[Install]
WantedBy=multi-user.target
--------------------------
systemctl start java.service
                                                                                                                                                                                                                                                                                                                                              
```