---
title: docker部署react
date: 2025-03-25 14:33:41
categories: react
tags: 
 - react
---

React项目docker部署，nginx配置

1. 项目下面新建Dockerfile文件，内容如下
```
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
RUN npm install -g serve
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["serve", "-s", "build", "-l", "3000"]
```
2. 编译镜像
```
拉取指定架构的基础镜像编译
docker build --platform linux/amd64 -t pt20011203/echo-admin:amd64 .
```
3. 上传到远程仓库
```
docker push pt20011203/echo-admin:amd64
```
4. 服务器拉取，并运行
```
docker pull pt20011203/echo-admin:amd64
docker run -d -p 3000:3000 --name echo-admin pt20011203/echo-admin:amd64
```
5. nginx配置，修改配置文件（这里有个小坑，要配置home-page与/echo-admin/一致，不然静态资源无法渲染）
```
location /echo-admin/ {
       proxy_pass http://127.0.0.1:3000/;
   }
```
   效果：https://www.techkid.top/echo-admin/login
 