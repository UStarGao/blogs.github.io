---
layout: post
title: Docker mirror speed
---
Docker默认镜像源在国外，国内下载速度可能略慢，配置国内镜像源，会大大提高Docker镜像下载速度。
-  配置 163 镜像加速器

[root@ansible ~]# vim /etc/docker/daemon.json
```
{
  "registry-mirrors":["http://hub-mirror.c.163.com"]
}
```
- 配置Docker国内镜像源

```
{
  "registry-mirrors": [ "https://registry.docker-cn.com"]
}
```

Author:UStarGao
