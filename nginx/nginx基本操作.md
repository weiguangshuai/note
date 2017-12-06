# nginx基本操作

## 安装、启动和停止
1·安装依赖和第三方库
>yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel

2·解压，编译
>tar zxf nginx-x.x.x.tar.gz\
cd dir\
解压完以后进行编译\
make\
编译完后进行安装\
make install
