<div style="text-align: center; font-weight: 700; font-size: 2em;">mysql安装及配置</div>

# 一、下载地址
```
https://dev.mysql.com/downloads/mysql/
```

# 二、使用别名
```
# 从命令行访问常用程序（如 mysql 和 mysqladmin）
alias mysql=/usr/local/mysql/bin/mysql
alias mysqladmin=/usr/local/mysql/bin/mysqladmin
```

# 三、添加系统环境变量
```
cd ~
vim .bash_profile
```
添加以下代码:
```
export PATH=${PATH}:/usr/local/mysql/bin
```
退出后
```
source .bash_profile
```
如果不进行以下操作的话，每打开一个终端，都要再输一遍 source .bash_profile
```
vim .zshrc
```
添加以下代码:
```
export PATH=${PATH}:/usr/local/mysql/bin
```
退出后
```
source .zshrc
```
# 四、修改密码
```
mysqladmin -u root -p password 12345678
```
12345678是我的新密码，自行修改成自己想要设置的密码

按回车后，提示输入密码，此时让输入的密码不是你电脑的密码，而是安装时设置的数据库密码。

# 五、登陆数据库
```
mysql -u root -p
```

# 六、彻底删除数据库
```linux
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*

vim /etc/hostconfig  (and removed the line MYSQLCOM=-YES-)
```