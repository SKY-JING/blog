<div style="text-align: center; font-weight: 700; font-size: 2em;">mysql安装</div>

# 一、检查mariadb
检查是否已安装过mariadb，若有便删除(命令系统自带)
```
[root@localhost /]# rpm -qa | grep mariadb
[root@localhost /]# rpm -e --nodeps mariadb-libs-5.5.44-2.el7.centos.x86_64
```

# 二、检查mysql
检查是否已安装过mysql，若有便删除
```
[root@localhost /]# rpm -qa | grep mysql
[root@localhost /]# rpm -e –-nodeps mysql-libs-5.1.52.x86_64
```

# 三、检查mysql组和用户
检查mysql组和用户是否存在，如无则创建
```
[root@localhost ~]# cat /etc/group | grep mysql
[root@localhost ~]# cat /etc/passwd |grep mysql
[root@localhost ~]# groupadd mysql
[root@localhost ~]# useradd -r -g mysql mysql
```

# 四、下载安装mysql
切换到/usr/local/src目录从官网下载mysql安装包，解压后移动到/usr/local/mysql下
```
[root@localhost src]# wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
[root@localhost src]# tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
[root@localhost src]# mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql
```

# 五、添加data目录
在mysql下添加data目录
```
[root@localhost ~]# mkdir /usr/local/mysql/data
```

# 六、更改权限
更改mysql目录下所有的目录及文件夹所属组合用户
```
[root@localhost /]# cd /usr/local/ 
[root@localhost local]# chown -R mysql:mysql mysql/
[root@localhost local]# chmod -R 755 mysql/
```

# 七、初始化mysql
编译安装并初始化mysql，记住命令行末尾的密码
```
[root@localhost local]# /usr/local/mysql/bin/mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
```
> 注：<br>错误：error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory<br>解决方案：yum install -y libaio

# 八、启动mysql
```
[root@localhost local]# /usr/local/mysql/support-files/mysql.server start
```

# 九、制作重启服务软链接
```
[root@localhost local]# ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
[root@localhost local]# service mysql restart
```

# 十、制作安装目录软链接
```
[root@localhost local]# ln -s /usr/local/mysql/bin/mysql /usr/bin
```

# 十一、登录msyql
输入密码（密码为步骤7初始化生成的密码）
```
[root@localhost local]# mysql -u root -p
Enter password:
```

# 十二、修改初始密码并开放远程
```
msql>alter user 'root'@'localhost' identified by '123456';
mysql>use mysql;
msyql>update user set user.Host='%' where user.User='root';
mysql>flush privileges;
mysql>quit
```
> 注：123456为新密码

# 十三、修改密码
```
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your password' WITH GRANT OPTION; 
mysql>flush privileges;
```
> 注：your password为新密码

# 十四、添加配置文件
编辑my.cnf，添加配置文件，配置内容为
```
[root@localhost local]# vi /usr/local/mysql/my.cnf
[mysqld]
port = 3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

# 十五、设置开机自启动
```
1、将服务文件拷贝到init.d下，并重命名为mysql
[root@localhost local]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
2、赋予可执行权限
[root@localhost local]# chmod +x /etc/init.d/mysqld
3、添加服务
[root@localhost local]# chkconfig --add mysqld
4、显示服务列表
[root@localhost local]# chkconfig --list
5、重启服务器
[root@localhost local]# reboot
```