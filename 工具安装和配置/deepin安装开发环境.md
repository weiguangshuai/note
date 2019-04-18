# 开发工具的安装

## deepin下安装jdk

1.下载jdk安装包，解压至安装目录中
>tar jdk.1.8.tar.gz\
mv jdk.1.8 /usr/local/java


2.配置jdk环境变量
>sudo vim /etc/profile

在文件后面加入下面配置
>JAVA_HOME=/usr/local/jdk8\
JRE_HOME=/usr/local/jdk8/jre\
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib\
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin\
export JAVA_HOME JRE_HOME CLASS_PATH PATH

执行命令让profile文件生效

>source /etc/profile

`使用deepin时应该注意，需要重启才能生效，不像ubuntu那样直接生效，这点需要注意`


## deepin下安装maven

1.下载maven安装包，解压至安装目录
>tar maven.tar.gz\
mv maven /usr/local/maven

2.配置maven环境变量
>sudo vim /etc/profile

在文件后面添加下面配置

>MAVEN_HOME=/usr/local/maven-3.5.0\
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin

3.更改maven的本地仓库位置
```xml
<settings>
    <localRepository>D:\maven_new_repository</localRepository>  
</settings>  
```

4.更改maven中jdk版本

```xml
<profile>    
    <id>jdk-1.8</id>    
     <activation>    
          <activeByDefault>true</activeByDefault>    
          <jdk>1.8</jdk>    
      </activation>    
   <properties>    
    <maven.compiler.source>1.8</maven.compiler.source>    
    <maven.compiler.target>1.8</maven.compiler.target>    
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>    
   </properties>    
</profile>
```

## deepin下安装tomcat

1.下载tomcat安装包，解压至制定目录
```
tar zxvf tomcat-8.0.tar.gz
mv tomcat-8.0 /usr/local/tomcat8
```

2.配置tomcat运行环境变量
```
打开startup.sh，在后面添加jdk和jre环境变量
sudo vim startup.sh
添加内容如下：
JAVA_HOME=/usr/local/jdk8
JRE_HOME=$JAVA_HOME/jre 
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME CLASSPATH=.:$JRE_HOME/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
TOMCAT_HOME=/usr/local/apache-tomcat-8.5.12
```

## deepin下安装mysql

1.直接使用apt-get包管理器安装mysql
```
sudo apt-get install mysql-server mysql-client
```
安装过程中需要自己设定root的密码

2.修改mysql为utf8编码
- 打开/etc/mysql/mysql.conf.d下的mysqld.cnf文件,修改如下
    - 在[mysqld] 的skip-external-locking下，添加character-set-server=utf8

- 打开/etc/mysql/conf.d下的mysql.cnf文件，添加如下内容
    - 在[mysql]下插入一行：default-character-set=utf8

3.为平时开发创建一个用户
```
create user 'username'@'host' identified by 'passwd'
```
- host表示该用户可以从哪个主机进行登录，如果所有允许所有主机进行登录，则使用通配符%

为用户授权
```
grant privileges on databasename.tablename to 'username'@'host'
```
- privileges：用户的操作权限,如select、update等，如果要授权所有权限则使用all
- databasename：数据库名
- tablename：表明

