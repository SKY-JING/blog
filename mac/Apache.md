<div style="text-align: center; font-weight: 700; font-size: 2em;">服务器配置</div>

# 一、启动 Apache 服务

1）终端命令：`sudo apachectl start`

2）浏览器输入：`http://localhost或http://127.0.0.1`

3）显示`It works！`表示启动成功

>注意：在启动Apache服务时，会提示输入开机密码。

# 二、查看Apache版本

终端命令：`sudo apachectl -v`

# 三、关闭Apache服务

终端命令：`sudo apachectl stop`

# 四、重启Apache服务

终端命令：`sudo apachectl restart`

# 五、Apache安装路径

默认路径：`/private/etc/apache2/`

打开方式：
1）终端输入：`open /etc`，找到 `etc` 文件夹下的 `apache2` 文件夹

2）Finder：使用快捷键 `command + shift + G` 打开前往文件夹输入框，输入 `/private/etc/apache2`

# 六、Apache应用部署路径

默认路径：`/Library/WebServer/Documents/`

修改默认路径：
在 `/private/etc/apache2/` 目录下找到并打开 `httpd.conf` 文件，搜索 `DocumentRoot` 修改部署路径

# 七、Apache端口

默认端口：`80`

修改默认端口：
在 `/private/etc/apache2/` 目录下找到并打开 `httpd.conf` 文件，搜索 `Listen 80` 修改端口号

# 八、Apache开启PHP服务

1）在 `/private/etc/apache2/` 目录下找到并打开 `httpd.conf` 文件

2）搜索 `php`，将前方的 `#` 删除

3）重启 `Apache` 服务即可

4）在应用部署路径下新建test.php（内容为：`<?php echo "我的第一段 PHP 脚本！"; ?>`），浏览器输入 `http://localhost`