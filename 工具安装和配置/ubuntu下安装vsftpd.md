# ubuntu下安装vsftpd

1、安装vsftpd
>sudo apt-get install vsftpd
	
2、新建目录/home/ftp作为用户的主目录，新建用户，更改密码；添加权限

	>sudo mkdir /home/ftp
	sudo useradd -d /home/ftp -M weigs
	sudo passwd weigs
	sudo chmod 777 /home/ftp


3、新建文件/etc/vsftpd.user_list，用于存放访问ftp的用户，在其中添加访问weigs用户

>sudo vim /etc/vsftpd.user_list

4、编辑vsftpd配置文件

>sudo vim /etc/vsftpd.conf\
>禁止匿名访问\
>anonymous_enable=NO\
>接受本地用户\
>local_enable=YES\
>允许上传\
>write_enable=YES\
>添加如下的注释：\
>userlist_file=/etc/vsftpd.user_list\
>userlist_enable=YES\
>userlist_deny=NO

5、启动并测试
>sudo service  start
>在命令行输入ftp ip即可访问

6、vsftpd相关命令
>sudo service start
>sudo service stop
>sudo service restart

7、注意要打开21端口，查看防火墙此端口是否打开