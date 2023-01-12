---
title: K8S常见指令   
date: 2021-09-07 16:20:46   
tags:
  - K8S
  - Docker
categories: K8S
excerpt: 分享一些K8S和Docker实用小技巧，走过路过，不要错过嘛.
---

## K8S
```shell
$ kubectl == kubernetes + control  # 即：k8s控制器
$ kubectl get namespace  # 查询物理cluster下拆分出的namespace信息
$ kubectl get node  # 查看k8s集群有哪些k8s节点
$ kubectl get pod -ndevga -owide -w |grep -i secrets-distribution-admin
$ kubectl -ndevga describe deploy secrets-distribution-admin-devga
$ kubectl get po pod-redis -o yaml  # 以yaml文件形式显示一个pod的详细信息
$ kubectl get po <podname> -o json  # 以json形式显示一个pod的详细信息
$ kubectl describe po rc-nginx-3-l8v2r
$ kubectl delete -f rc-nginx.yaml   
$ kubectl delete po rc-nginx-btv4j   
$ kubectl delete po -lapp=nginx-2
$ kubectl -n prodbj exec nginx-2476590065-1vtsp  -it sh
$ kubectl get service
$ kubectl get service nginx -o yaml > nginx_forreplace.yaml
$ kubectl -n qa rollback status deployment xxxxxxx
$ kubectl -n qa rollback resume deployment xxxxxxx
$ kubectl rollout history deployment cargo-detail-dev -n dev  # 滚动发布的历史记录
$ kubectl rollout undo deployment cargo-detail-dev -n dev --to-revision=7  # 回滚到指定的某个版本

$ kubectl rollout restart deploy xxxxx
$ kubectl -n qa get crd  # 获取所有的自定义资源

# node节点打标
$ kubectl -n qa lable nodes ip/name key=value  # 加标签
$ kubectl -n qa lable nodes ip/name key=value --overwrite  # 更新
$ kubectl -n qa lable nodes ip/name key-  # 删除标签
$ kubectl -n qa edit node ip/name  # 编辑node资源   -- 看起来node节点可以同时打多个标签，倒也合理
```

## Docker
```shell
#!/bin/env bash

# docker stop jenkins
# docker rm jenkins
$ docker run -d \
    --name jenkins \
    -p 18101:8080 \
    -p 50000:50000 \
    -v /srv/jenkins:/var/jenkins_home \
    -v /usr/local/maven:/usr/local/maven \
    -v /usr/local/jdk:/usr/local/jdk  \
    -v /etc/localtime:/etc/localtime \
    -v /etc/timezone:/etc/timezone \
    -u root \
    jenkins/jenkins:latest
  #  --link gitlab:47.97.174.90 \
```

```shell
#!/bin/env bash
$ host_name=gitlab.virtual.vm
$ gitlab_dir=/srv/gitlab
$ docker stop gitlab
# docker rm gitlab
$ docker run -d \
  --hostname ${host_name} \
  -p 8443:443 -p 1080:80 -p 2222:22 \
  --name gitlab \
  --restart always \
  --env gitlab_rails['SIDEKIQ_MEMORY_KILLER_MAX_RSS']=2048 \
  -v ${gitlab_dir}/config:/etc/gitlab \
  -v ${gitlab_dir}/logs:/var/log/gitlab \
  -v ${gitlab_dir}/data:/var/opt/gitlab \
  -v /var/run/docker.sock:/run/docker.sock \
  registry.cn-hangzhou.aliyuncs.com/imooc/gitlab-ce:latest
```

```shell
$ docker image inspect image-name
$ docker inspect container-name
$ docker port container-name
$ docker kill/start/stop container-name
$ docker pull jenkins打包日志里的镜像名
$ docker run --name cargo-test -u root --rm -P  harbor.test.com/app/cargo-test:python-dev-jdk8-20200109_161946 python3 -c "import schedule"
$ docker run -it --name testmock -u root --rm -P harbor.test.com/app/testmock:python-master-jdk8-20200407_143644 bash
$ docker run -it --entrypoint /bin/bash --name bapis-pre -u root --rm -P hub.test.co/k8s-prow/bapis-pre:v0.0.7
```

```shell
# 常用docker命令
# 1、删除所有镜像
$ docker rmi $(docker images -q) -f
# 2、删除所有容器
$ docker stop $(docker ps -q) & docker rm $(docker ps -aq)

# 查看所有容器： 
$ docker ps -a
 
# 查看运行容器：
$ docker ps
 
# 停用所有容器：
$ docker stop $(docker ps -q)
 
# 删除所有容器：
$ docker rm $(docker ps -aq)
 
# 停用和删除所有容器：
$ docker stop $(docker ps -q) & docker rm $(docker ps -aq)

# 镜像改名，便于本地镜像推到远端
$ docker tag [镜像id] [远程ip:端口/自定义路径/*]:[版本号]
$ docker push [远程ip:端口/自定义路径/*]:[版本号]
```
