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


