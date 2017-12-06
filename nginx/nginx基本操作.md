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


