<div style="text-align: center; font-weight: 700; font-size: 2em;">git安装及配置</div>

# 一、下载
```
https://git-scm.com/downloads
```
# 二、验证是否成功
```
git --version
```
# 三、添加git账户和邮箱
```
git config --global user.name "你的注册用户名"
git config --global user.emall "你的注册邮箱"
```
# 四、密钥

1）检查电脑是否曾经生成过秘钥：
```
cd ~/.ssh
```
若打开该文件夹为空，则表示没有生成过秘钥。

2）生成秘钥
```
ssh-keygen -t rsa -C "your_email@youremail.com"
```
执行成功后，会在主目录`.ssh`路径下生成两个文件：`id_rsa`私钥文件；`id_rsa.pub`公钥文件

> 命令要求输入密码，不用输，三个回车即可

3）在远程仓库`gitlab`上添加`title`和`key`:
```
Settings -> 左栏点击 SSH and GPG keys -> 点击 New SSH key
```
`title`可以自己取一个容易区分的名字，`key`为`id_rsa.pub`中的内容（全部复制，可用`cat id_rsa.pub`命令打开）

# 五、验证
```
ssh -T git@github.com
```
正确输出：`Hi xxx! You've successfully authenticated, but GitHub does not # provide shell access.`

> 如果提示`Are you sure you want to continue connecting (yes/no)?`，直接输入yes