## 代理

### 正向代理

- resolver address ... [valid=time]：该指令用于指令DNS服务器的IP地址
    - address：DNS服务器的ip地址
    - time：设置数据包在网络中的有效时间

- resolver_timeout time：用与设置DNS服务器域名解析超时时间

- proxy_pass URL：用于设置代理服务器的协议和地址，更主要的是应用在反向代理


```
...
server
{
    resolver 8.8.8.8;
    listen 82;
    location /
    {
        proxy_pass http://www.weigs.com
    }
}
...

```
注意：**一般代理服务器配置在一个server块中，在该server块中，不要出现server_name指令，resovler指令是必须的，没有该指令，nginx服务器无法处理接收到的域名**

### 反向代理

- proxy_pass RUL：设置被代理服务器的地址
如果被代理服务器是一组地址的话，可以使用upstream指令配置

```
upstream proxy_svrs
{
    server http://127.0.0.1:8080/uri/;
    server http://127.0.0.1:81/url/;
}
server
{
    listen 80;
    location /
    {
        proxy_pass proxy_svrs;
    }
}
```
注意：**在使用该指令时应该注意，url中是否包含uri。如果url不包含uri，不会改变原地址的uri；如果包含uri，服务器将会使用新的uri替代原来的uri**
```
server
{
    listen 80;
    server_name www.myweb.name;
    location /server/
    {
        proxy_pass http://127.0.0.1;
    }
}

在客户端使用”http://www.myweb.name/server“发送请求时，由于proxy_pass指令的url不包含uri，所以转到"http://127.0.0.1/server"
```

- proxy_pass_request_body on | off：该指令用于配置是否将客户端请求的请求体发送给代理服务器

- proxy_set_header field value：该指令可以更改nginx服务器接收到的客户端请求的请求头信息，然后将新的请求头发送给被代理的服务器
    - field：要更改的信息所在的头域
    - value：更改的值

- proxy_set_body value：该指令可以更改nginx服务器接收到的客户端请求的请求体信息，然后将新的请求体发送给被代理的服务器

- proxy_redirect redirect repalcement：该指令用于修改被代理服务器返回的响应头中的location头域；
    - redirect：匹配location头域值的字符串，支持变量的使用和正则表达式
    - replacement：用于替代redirect变量内容的字符串，支持变量的使用
```
location: http://localhost:8080/proxy/some/uri/
指令设置：
proxy_redirect http://localhost:8080/proxy/ http://myweb/frontend/;
location的头域信息将变为：
http://myweb/fronted//some/uri/
```

- proxy_intercept_error on | off：开启状态时，如果被代理的服务器返回的http状态码为400或者大于400，则nginx服务器使用自己定义的错误页；如果关闭，nginx服务器直接将被代理服务器返回的http状态返回给客户端

- proxy_ssl_session_reuse on | off：该指令用于配置是否使用基于ssl安全协议的回话连接被代理的服务器

- proxy_cache zone | off：配置一块公共的内存区域，该区域可以存放缓存的索引数据
    - zone：设置用于存放缓存索引的内存区域的名称
    - off：关闭proxy_cache功能，默认设置

- proxy_cache_bypass string ...：该指令用于配置nginx服务器向客户端发送数据时，不从缓存中获取的条件。

- proxy_cache_key string：设置nginx服务器在内存中为缓存数据建立索引时使用的关键字

- proxy_cache_path path [levels=levels] keys_zone=name:size1 [inactive=time1] [max_size=size2] [loader_files=number] [loader_sleep=time2] [loader_threshold=time3] ： 该指令用于设置nginx服务器存储缓存数据的路径以及和缓存索引有关的内容
    - path：设置缓存数据存放的路径
    - levels：设置在相对于path指定目录的第几级hash目录中缓存数据
    - name：size1 ： 用来设置存放缓存索引的内存区域的名称和大小
    - time1：设置强制更新缓存数据的时间
    - size2：设置硬盘中缓存数据的大小限制

## 负载均衡

- 实例：对所有请求实现一般轮旬规则
```
upstream beckend
{
    server 192.168.1.2:8080;
    server 192.168.1.3:80;
    server 192.168.1.4:8080;
}

server
{
    listean 80;
    server_name www.myweb.name;
    index index.html index.htm;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }
}
```

- 实例：对所有请求实现加权轮询规则

```
upstream backend
{
    server 192.168.1.2:80 weight=5;
    server 192.168.1.2:80 weight=2;
    server 192.168.1.2:80;
}

server
{
    listen 80:
    server_name www.myweb.name;
    index index.html index.htm;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }
}
```

- 实例：对特定资源实现负载均衡

```
upstream videobackend
{
    server 192.168.1.1:80;
    server 192.168.1.2:80;
    server 192.168.1.3:80;
}

upstream filebackend
{
    server 192.168.1.4:80;
    server 192.168.1.5:80;
    server 192.168.1.6:80;
}

server 
{
    listen 80;
    server_name www.myweb.name;
    index index.html index.htm;
    location /video/ {
        proxy_pass http://videobackend;
        proxy_set_header Host $host;
    }

    location /file/ {
        proxy_pass http://filebackend;
        proxy_set_header Host $host;
    }
}
```

- 对不同域名实现负载均衡

```
upstream bbsbackend
{
    server 192.168.1.1:80 weight=2;
    server 192.168.1.2:80 weight=2;
    server 192.168.1.3:80;
}

upstream homebackend
{
    server 192.168.1.4:80;
    server 192.168.1.5:80;
    server 192.168.1.6:80;
}

server
{
    listen 80;
    server_name home.myweb.name;
    index index.html index.htm;
    location / {
        proxy_pass http://homebackend;
        proxy_set_header Host $host;
    }
}

server
{
    listen 81;
    server_name bbs.myweb.name;
    index index.html index.htm;
    location / {
        proxy_pass http://bbsbackend;
        proxy_set_header Host $host;
    }
}
```