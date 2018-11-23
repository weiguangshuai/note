# Docker

## 安装Docker

> Docker安装在centos7上，但是要注意的是centos系统的内核必须是3.10及以上的版本，不然安装不成功

- 首先是确定系统是否能够安装Docker

```shell
#查看系统的全部信息
uname -a
#查看系统的内核信息
uname -r
```

- 卸载旧的版本（如果安装过旧的版本话）

```shell
yum remove docker  docker-common docker-selinux docker-engine
```

- 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 设置yum源

```shell
#注意：如果已经设置了源，想更改源的时候报错，可以进入/etc/yum.repos.d目录中删除docker-ce.repo文件
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
#或者使用阿里云的镜像
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 查看仓库中的所有Docker版本，并选择特定的版本安装

```shell
yum list docker-ce --showduplicates | sort -r
```

- 安装Docker

```
#默认安装仓库中最新的版本
yum install docker-ce
#指定版本安装docker，<FQPN>表示指定的版本
yum install <FQPN>
```

- 修改/etc/docker目录下的daemon.json文件（如果没有需要自己创建），在文件中加入以下的内容

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

- 修改配置文件后需要重启容器（相关命令如下所示）

```shell
#启动容器
service docker start
#重启容器
service docker restart
#停止容器
service docker stop
```



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

- `docker build -t <镜像名> <Dockerfile路径>`

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

- `docker login --username=[username] --password=[password] [registryURL]`：登录到某个远程仓库

- `docker tag`：标记本地镜像，将其归入某一仓库

- `docker push [remoteURL]：[imageTag]`：推送到一个镜像到中心仓库

- `docker create -it --name test daocloud.io/library/ubuntu`：创建一个容器命名为 test 使用镜像daocloud.io/library/ubuntu

- `docker rm [容器ID]`：删除一个容器

  - docker rm \`docker ps -a -q`：删除所有的容器

- `docker commit <CONTAIN-ID> <IMAGE-NAME>`：把一个正在运行的容器保存为镜像

  ​
##Dockerfil文件

Dockerfile由两个部分组成，注释和命令+参数，下面是构建Springboot项目时使用的Dockerfile：

```dockerfile
FROM java:8
ADD spring-boot-docker-1.0.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

### 常用命令

- `FROM`：获取基础镜像，FROM必须是第一个命令，如果需要多个镜像时，可以使用多个FROM指令（每个镜像一次）

```dockerfile
# FROM [image name]
FROM ubuntu
FROM java:8
```

- `MAINTAINER`：维护者信息，已经被弃用
- `RUN`：在构建镜像过程中执行特定的指令，并生成一个中间镜像
  - `RUN <commond>`：shell格式
  - `RUN ["executable", "param1", "param2"]`：exec格式
- `CMD`：指定容器运行时的默认参数，如果出现多次以最后一次为准
  - `CMD ["executable", "param1", "param2"]`：exec格式
  - `CMD command param1 param2`：shell格式
- `LABEL`：给构建的镜像打标签，如果base image中也有标签，则继承，如果是同名标签，则覆盖。
  为了减少图层数量，尽量将标签写在一个`LABEL`指令中去

```dockerfile
LABEL <key>=<value> <key>=<value> ...
# 实例
LABEL multi.lable1="value1" multi.lable2="value2" other="value3"
```

- `EXPOSE`：为构建的镜像设置监听端口，但是并不会让容器监听host的端口

```dockerfile
EXPOSE <port> [<port>...]
#多个映射接口
EXPOSE 8080
EXPOSE 8090
EXPOSE 9090
```

- `ENV`：在构建的镜像中设置环境变量

```dockerfile
#可以设置多个环境变量，如果<value>中存在空格，需要转义或用引号"括起来
ENV myName="John Doe" \
    myDog=Rex\ The\ Dog \
    myCat=fluffy
```

- `ADD`：在构建镜像时，复制文件到镜像中

```dockerfile
# <src>可以是文件、目录，也可以是文件URL,如果<src>是个目录，则复制的是目录下的所有内容，但不包括该目录
# <dest>可以是绝对路径，也可以是相对WORKDIR目录的相对路径
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

- `COPY`：和`ADD`命令类似
- `ENTRYPOINT`：指定镜像的执行程序，只有最后一条的`ENTRYPOINT`指令有效

```dockerfile
#CMD和ENTRYPOINT至少得使用一个。ENTRYPOINT应该被当做docker的可执行程序，CMD应该被当做ENTRYPOINT的默认参数
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

- `VOLUME`：指定镜像内的目录为数据卷，使此目录具有持久化存储数据的功能，该目录可以被容器本身访问，也可以被其他容器访问

```dockerfile
VOLUME ["/var/log"]
VOLUME /var/log /var/db
```



## 部署前端项目

> 前端部署需要安装nginx镜像，通过将硬盘中的目录和配置文件挂载到容器中的文件来实现项目的启动，具体操作如下：

1、第一种方法使用nginx镜像启动，启动时挂载主机的硬盘，读取硬盘中的文件启动nginx，具体如下：

- 拉取nginx官方镜像

```shell
#从官方库中拉取nginx镜像
docker pull nginx
```

- 使用nginx镜像运行容器

```shell
# -p 80:80：将容器80端口映射到主机的80端口
# -v $PWD/nginx.conf:/etc/nginx/nginx.conf：将冒号前面的主机目录挂载到冒号后面的容器目录，此处是启动nginx的配置文件
# -v $PWD/html:/etc/nginx/html：同上，此处是将项目所在主机的目录挂载到容器中
docker run -p 80:80 --name weigs-ngnix -d -v $PWD/nginx.conf:/etc/nginx/nginx.conf -v $PWD/html:/etc/nginx/html nginx
```

2、第二种方法是直接将nginx和前端代码打包成docker镜像，具体操作如下：

- Dockerfile文件内容

```dockerfile
FROM nginx
#删除容器中nginx的默认配置文件
RUN rm -rf /etc/nginx/nginx.conf
#将本地代码copy到容器中
ADD mz-feature-mwc-customization /etc/nginx/html/mz-feature-mwc-customization
#使用本地自己配置的nginx配置文件代替容器中默认的配置文件
ADD nginx.conf /etc/nginx/nginx.conf
```

- 构建镜像

```shell
docker build -t weigs/xxxx .
```

- 启动镜像进行测试

```dockerfile
#注意：容器内部的映射端口应该和nginx配置文件的启动端口一样
docker run -p 8080:8080 --name my-test -d weigs/xxxx
```

