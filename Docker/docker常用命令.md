# Docker 常用命令
## 整体信息
```bash
docker info
```
## 释放空间
1. `docker system prune`(慎用): 删除
   1. 已经停止的容器；
   2. 未被使用的网络；
   3. 所有未打标签的镜像；
   4. 构建镜像时产生的缓存
2. `docker container prune`: 删除已经停止的容器
3. `docker network prune`: 删除未被使用的网络
4. `docker image prune`: 删除未打标签的镜像
5. `docker image prune -a`: 删除没有容器的镜像
6. `docker volume prune`: 删除未被使用的数据卷
## 过滤
1. `docker ps -f id=xxxxx`
2. `docker ps -f name=xxxxx`
3. `docker ps -f status=[running, created, restarting, running, removing, paused, exited, dead]`
## 信息获取
1. `docker ps -s`: 查看容器所占硬盘空间，0 为容器自身所占资源大小，406 为镜像+容器大小
![avator](./img/docker%20ps%20-s.png)
2. `docker ps --no-trunc`: 查看完整信息