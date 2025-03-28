---
title: 使用 Jenkins 进行 CI CD 流程
date: 2024-12-16T10:35:38+08:00
updated: 2024-12-16T14:00:46+08:00
dg-publish: false
---

## 使用 Docker 安装 Jenkins

推荐的镜像 <https://hub.docker.com/r/jenkinsci/blueocean/>

使用

```sh
$ docker pull jenkinsci/blueocean
$ docker run -p 49000:8080 -p 50000:50000 -d -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

之后访问 `http://IP:49000`

## 前端部署

中间层

## 后端部署
