# nginx基本操作

## 安装、启动和停止

### 安装

1·安装依赖和第三方库
>yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel

2·解压，编译
>tar zxf nginx-x.x.x.tar.gz\
cd dir\
解压完以后进行编译\
make\
编译完后进行安装\
make install

### 启动、重启和停止

1·启动

启动时可以使用一些参数，这些参数可以显示一些需要的内容，比如`-h`用来打印二进制文件nginx的用法
>./xxx/nginx -h

2·重启

nginx的平滑重启
>./xxx/nginx -g HUP [-c newConfigFile]

- HUP用于发送平滑重启信号
- newConfigFile，可选项，用于指定新配置文件的路径

3·停止

- 快速停止
>./sbin/nginx -g TERM | INT

- 平缓停止
>./sbin/nginx -g QUIT


## 配置文件

### 文件结构

```
    ...                                 #全局块
    events                              #events块
    {
        ...
    }
    https                               #http块
    {
        ...                             #http全局块
        server                          #server块
        {
            ...                         #server全局块
            location [pattern]          #location块
            {
                ...
            }
            location [pattern]          #location块
            {
                ...
            }
        }

        server                          #server块
        {
            ...
        }
        ...                             #https全局块
    }
```

- 全局块：主要是设置一些影响nginx服务器整体运行的配置指令，这些指令作用域是nginx服务器全局

    - user user [group]:用于设置用户或者用户组。只有设置了的用户或者用户组才能启动服务器，否则就会出错；如果希望任何用户都可以启动，有两种方法：
        - 将此行注释掉
        - 将用户设置魏nobody


    - work_processes：wrok_processess number | auto；
        - number：指定服务器最多可以产生的work process数
        - auto：nginx将自动检测


    - error_log：错误日志的存放路径，参数包括存放路径和日志的级别
        - error_log logs/error.log error：错误级别的日志存放路径

    - include file：配置文件的引入，支持相对路径

- event块：涉及nginx服务器与用户的网络连接
    - accept_mutex on | off：对多个nginx进程进行序列化，防止多个进程对连接的争抢
    - mutil_accept on | off：设置是否同时接受多个网络连接，默认off
    - worker_connections number：设置每个work_process同时开启的最大连接数


- http块：这是nginx最重要的一块，代理、缓存和日志定义等绝大多数功能都在这个模块中

    - MIME-Type：服务器能够识别的文件格式
    - access_log path [format[buffer=size]]：自定义服务日志，用于记录nginx服务器提供服务过程中应答前端请求的日志
        - path：存放路径
        - format：服务日志的格式
        - size：临时存放日志的内存缓存区大小
        - access_log off取消服务日志功能

    - keepalive_timeout timeout[header_time]：连接超时时间、
        - timeout：服务器对连接的保持时间
        - header_timeout：可选项，在应答报文头部的Keep_Alive域设置超时时间
        - 可以在http块、server块、location块配置
    - keepalive_requests number：单连接请求次数上限

- server块：和“虚拟主机”的概念有密切联系，每个server块相当于是一个虚拟主机
    - listen prot：配置网络监听
    - server_name name ...：基于名称的虚拟主机配置
        - name：可以由一个组成，也可以多个并列，之间用空格隔开
        - 对于一个名称被多个虚拟主机的server_name匹配成功，优先级选择规则如下
            - 准确匹配server_name
            - 通配符在开始时匹配server_name成功
            - 通配符在结尾时匹配server_name成功
            - 正则表达式匹配server_name成功
- location块：server中可以包含多个location块，location相当于是server块的一个指令
    - location语法结构：location [= | ~ | ~* | ^~] uri {......}
    - alias path：更改location的url
    - error_page code ... [=[response]] uri:设置网站的错误页面
        - code：要处理的http错误代码
        - response：将code转化为新的错误代码response
        - uri：错误页面的路径或者网址



## nginx服务器的进程
>到目前为止，nginx服务器有三大类进程：一类是主进程，一类是由主进程生成的工作进程，还有就是用于缓存文件建立索引的进程

1.主进程：主要功能是与外界通信和对内部其他进程进行管理
- 读取nginx配置文件并验证其有效性和正确性
- 建立、绑定和关闭socket
- 按照配置生成、管理和结束工作进程
- 接收外界指令，比如重启、升级和退出等
- 开启日志文件，获取文件描述符等

2.工作进程：由主进程生成，生成数量可以通过Nginx配置文件指定，正常情况下存在于主进程的整个生命周期

- 接受客服端请求
- 将请求依次送入各个功能模块进行过滤处理
- IO调用，获取相应数据
- 与后端服务器通信，接受后端服务器处理结果
- 数据缓存，访问缓存索引、查询和调用缓存数据
- 发送请求结果，响应客服端请求
- 接受主进程指令等

3.缓存索引重建及管理进程：在Nginx服务启动一段时间后由主进程生成，在缓存元素重建完成后自动退出

## nginx服务器gzip压缩

>gzip的相关指令可以在配置文件的http块、server块或者location块中设置

- gzip on | off：开启或关闭Gzip功能
- gzip_buffers number size：设置Gzip压缩文件使用缓存空间大小
    - number：指定nginx服务器需要向系统申请缓存空间的个数
    - size：指定每个缓存空间的大小
- gzip_comp_level level：设定Gzip压缩程度，包括级别1到9。数字越小表示压缩程度越低
- gzip_disable regex... :针对不同客服端请求，可以选择性地开启和关系gzip功能
- gzip_min_length length：根据相应页面的大小，选择性开启或者关闭gzip指令
- gzip_types mime-type ... ：根据响应页的MIME类型选择性开启Gzip压缩功能
- gunzip_static on|off：开启或者关闭该模块功能，解压缩


## Nginx后端服务器配置
- upstream:设置后端服务器组的主要指令
- server address [parameters]：该指令用于设置组内的服务器
    - address:服务器地址
    - parameter：当前服务器配置更多属性
        - weight = number：为组内服务器设置权重，权重值高的服务器被优先处理
        - max_fails = number：设置请求失败的次数


- ip_hash指令：实现会话保持功能，将某个客户端的多次请求定向到组内同一台服务器上。

- least_conn指令：使用负载均衡策略为网络连接分配服务器组内服务器，该指令在功能上实现了最少连接负载均衡算法。
```
upstream beckend
{
    least_conn;
    ip_hash;
    server 127.0.0.1 weight=50
    server 127.0.0.2 fail_timeout=20
}
```


## rewrite功能
>rewrite指令通过正则表达式的使用来改变uri，可以同时存在一个或者多个指令，按照顺序依次对url进行匹配和处理

- rewrite regex replacement [flag]
    - replacement，匹配成功后用于替换uri中被截取内容的字符串
    - flag，用来设置rewrite对url的处理行为
        - last，终止继续在本location块中处理接收到的uri，并将此处重写的uri作为一个新的uri
        ```
        location / {
            rewrite ^(/myweb/.*)/media/(.*)\..*$ $1/map3/$2.map3 last;
            rewrite ^(/myweb/.*)/audio/(.*)\..*$ $1/map3/$2.ra last;
        }
        如果某个uri在第一个rewrite匹配成功并处理，nginx将不再使用第二个rewrite进行处理新的uri，而是让所有的location块重新匹配和处理新的uri
        ```
        - break，将此处重写的uri作为一个新的uri，在本块中继续进行处理。该标示将重写后的地址在当前location块中执行，不会将新的uri转向到其他location块中
        ```
        location /myweb/ {
            rewrite ^(/myweb/.*)/media/(.*)\..*$ $1/map3/$2.map3 break;
            rewrite ^(/myweb/.*)/audio/(.*)\..*$ $1/map3/$2.ra break;
        }
        如果uri在第2行被匹配成功并处理，nginx服务器将新的uri继续在本location块中使用第3行进行匹配和处理
        ```

- rewrite_log on | off ：配置是否开启url重写日志的输出功能
注意：**rewrite接收到的url不包括host地址，因此，regex不可能匹配到uri的host地址**


#rewrite的使用

- 域名跳转
```
...
server
{
    listen 80;
    server_name jump.myweb.name;
    rewrite ^/ http://www.myweb.info/;
    ...
}

...
```

- 镜像域名

- 独立域名

- 目录自动添加"/"

- 目录合并

- 防盗链



