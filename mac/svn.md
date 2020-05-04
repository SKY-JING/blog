<div style="text-align: center; font-weight: 700; font-size: 2em;">svn配置</div>

# 一、新建svn配置存放地址
```
cd 任意路径
mkdir svn
svnadmin create svn/store
```

# 二、配置

## 2.1 直接打开 `store` 文件夹

## 2.2 打开 `svnserve.conf` 文件，把以下四行得＃号删掉并删除前面的空格直至顶头对齐
```
anon-access = read
auth-access = write
password-db = passwd
authz-db = authz
```

## 2.3 打开 `passwd` 文件，在 `[users]` 下新加用户名＝密码
```
[users]
testname1 = 123456
```

## 2.4 打开 `authz` 文件

在groups下新增:
```
[groups]
admin ＝ testname1
```

在文件结尾新增:
```
[/]
@admin = rw
```
## 2.5 重启svn服务
```
svnserve -d -r svn/store/
```

> 关闭svn服务，可以直接进入活动管理器搜索svnserve进行关闭