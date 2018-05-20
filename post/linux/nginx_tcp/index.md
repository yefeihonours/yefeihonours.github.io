## 安装Nginx
### 需要软件包

* nginx-1.10.3.tar.gz
* pcre-8.40.tar.gz
* nginx_tcp_proxy_module.zip

<!--more-->

### 解压软件
```
$ tar zxvf nginx-1.10.3.tar.gz
$ tar zxvf pcre-8.40.tar.gz
$ unzip nginx_tcp_proxy_module.zip
$ # 安装
$ cd nginx-1.10.3
$ patch -p1 < /path/to/nginx_tcp_proxy_module/tcp.patch
$ ./configure --with-pcre=/path/to/pcre-8.40  --add-module=/path/to/nginx_tcp_proxy_module
$ make
$ make install
```
## 添加TCP配置
```
$ vim /usr/local/nginx/conf/nginx.conf
```
### 在与http配置的同级目录下, 添加以下代码
```
tcp {
	server {
	listen 8988;
	proxy_pass 192.168.15.237:36000;
	}
}
```
### 启动nginx
```
$ /usr/local/nginx/sbin/nginx
$ # or 
$ # /usr/local/nginx/sbin/nginx –s reload
```
