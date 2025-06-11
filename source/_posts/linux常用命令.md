---
title: linux常用命令
date: 2025-04-21 21:30:50
categories:
 - linux
tags: 
 - linux
---

#### Linux常用命令
文件上传下载：
```
scp -i <private_key> <local_file> <user>@<server>:<remote_path>
scp -i <private_key> <user>@<server>:<remote_file> <local_path>
rsync -avz --progress -e "ssh -i ~/yqzh-server.pem" /home/user/bigfile.zip root@8.133.205.1:/root/Uploads/
rsync -avz --progress -e "ssh -i ~/yqzh-server.pem" root@8.133.205.1:/root/Uploads/bigfile.zip /home/user/downloads/

```

```
root@yuy-Nuvo-8208GC-Series:~/heygem_data# cat /etc/docker/daemon.json
  {
    "registry-mirrors": [
        "https://docker.1panel.dev",
                "https://docker.fxxk.dedyn.io",
                "https://docker.xn--6oq72ry9d5zx.cn",
                "https://docker.m.daocloud.io",
                "https://a.ussh.net",
                "https://docker.zhai.cm"
        ]
  }
```