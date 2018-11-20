# Docker

## Docker常用命令

### 镜像操作

- `docker images`：显示本地所有的镜像

- `docker rmi <容器Id>`：本地移除一个或多个镜像

  - docker rmi \`docker images -a -q\`

- `docker history [id]`：查看指定镜像的创建历史

- `docker save`：指定镜像保存为tar归档文件

  - docker save -o ubuntu14.tar ubuntu:14：将镜像ubuntu:14保存为ubuntu14.tar文件

- `docker load`：将tar文件导入为镜像

  - docker load -i ubuntu14.tar
  - docker load < /home/save.tar

- `docker build -t <镜像名> \<Dockerfile路径\>`

  - docker build -t weigs/demo .：weigs/demo代表镜像名，.代表当前路径

- `docker run [镜像名称]`：运行该镜像的容器

  - -d：后台运行容器，返回容器的ID
  - -h：指定容器的hostname
  - -p：容器端口和linux端口的映射，如：-p 8080：8090
  - -e：设置环境变量

- `docker search`：搜索远程仓库中的镜像

  - docker search -s 3 --automated --no-trunc django：搜索收藏数不小于 3 ，并且能够自动化构建的  django 镜像，并且完整显示镜像

- `docker pull ubuntu:latest`：拉取ubuntu最新的镜像

  ​

### 容器操作

- `docker ps`：查看当前运行的容器
- `docker ps -a`：查看全部的容器
- `docker ps -a -q`：查看全部容器的ID和信息
- `docker ps -as`：查看全部容器占用的空间


- `docker logs -f <容器名或者ID>`：查看容器日志

- `docker start|stop|restart [id]`：docker容器的启动、停止和重启，以id作为参数

- `docker pause|unpause [id]`：暂停、恢复某一个容器的进程

- `docker commit`：保存对容器的修改

  - -a：作者
  - -c：改变列表
  - -m：提交信息
  - -p：提交时暂停容器
  - docker commit -a "user" -m "commit info" \[CONTAINER] \[imageName]:[imageTag]

- `docker login --username=[username] --password=[password] \[registryURL]`：登录到某个远程仓库

- `docker tag`：标记本地镜像，将其归入某一仓库

- `docker push [remoteURL]：[imageTag]`：推送到一个镜像到中心仓库

- `docker create -it --name test daocloud.io/library/ubuntu`：创建一个容器命名为 test 使用镜像daocloud.io/library/ubuntu

- `docker rm [容器ID]`：删除一个容器

  - docker rm \`docker ps -a -q`：删除所有的容器

- `docker commit \<CONTAIN-ID> \<IMAGE-NAME>`：把一个正在运行的容器保存为镜像

  ​

