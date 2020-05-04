<div style="text-align: center; font-weight: 700; font-size: 2em;">nginx部署</div>

# 一、准备资源，下载nginx及相关插件
  * cd usr/local/src 进入下载目录（可自定义）
  * 下载下面四个程序
```linux
wget http://nginx.org/download/nginx-1.10.2.tar.gz
wget http://www.openssl.org/source/openssl-fips-2.0.10.tar.gz
wget http://zlib.net/zlib-1.2.11.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
// 如果未安装c++环境，先安装（yum install gcc-c++ 点击y即可）
```

# 二、安装nginx及其组件
## 2.1 安装openssl
```linux
tar -zxvf openssl-fips-2.0.10.tar.gz
cd openssl-fips-2.0.10
./config && make && make install 
```
## 2.2 安装pcre
```linux
tar -zxvf pcre-8.40.tar.gz
cd pcre-8.40
./configure && make && make install
```
## 2.3 安装zlib
```linux
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure && make && make install
```
## 2.4 安装nginx
```linux
tar zxvf nginx-1.10.2.tar.gz
cd nginx-1.10.2
./configure && make && make install
```

# 三、启动nginx
  * 查看nginx安装地址（whereis nginx）
  * 进入目录启动并启动 
```linux
cd /usr/local/nginx
/usr/local/nginx/sbin/nginx
```

# 四、查看安装是否成功
打开浏览器，通过服务器外网ip访问，出现nginx提示页面表示安装成功。