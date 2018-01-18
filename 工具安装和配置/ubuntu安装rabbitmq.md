# ubuntu安装rabbitmq

添加RabbitMQ创库源

```shell
sudo vim /etc/apt/sources.list
#在文件最后一行添加如下内容
deb http://www.rabbitmq.com/debian/ testing main
```

添加密匙

```shell
cd /tmp
wget https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
sudo apt-key add rabbitmq-signing-key-public.asc
```

安装RabbitMQ

```shell
sudo apt-get update
#直接安装，系统会自动安装缺少的erlang
sudo apt-get install rabbitmq-server
```

RabbitMQ开启与重启

```shell
sudo service rabbitmq-server start
sudo service rabbitmq-server restart
sudo service rabbitmq-server stop
```

启用RabbitMQ-management插件（这样可以在浏览器登录访问，访问ip:15672，默认用户和密码：guest）

```shell
sudo rabbitmq-plugins enable rabbitmq_management
#重启RabbitMQ
sudo service rabbitmq-server restart
```

