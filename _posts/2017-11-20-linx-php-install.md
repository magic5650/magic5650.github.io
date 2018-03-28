---
layout: post
title: php编译安装
author: magic
date:   2017-11-20
categories: Linux
tags: linux php
permalink: /archivers/linux-php-install
---
* 目录
{:toc}

## 安装必备的软件包
```
yum install \
bzip2 bzip2-devel \
curl curl-devel \
freetypefreetype-devel \
gcc gcc-c++ \
gd gd-devel \
gmp gmp-devel \
glib2 glib2-devel \
glibc glibc-devel \
libcurl libcurl-devel \
libjpeg libjpeg-devel \
libmcrypt libmcrypt-devel \
libpng libpng-devel \
libxml2 libxml2-devel \
libxslt libxslt-devel \
openssl openssl-devel \
pcre pcre-devel \
readline readline-devel \
zlib zlib-devel
```

<!--more-->

## 安装libiconv
```
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
tar zxvf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local/libiconv
make && make install
```
## 安装php7.2
```
wget http://am1.php.net/distributions/php-7.2.3.tar.gz
tar zxf php-7.2.3.tar.gz
cd php-7.2.3

./configure \
--prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-iconv-dir=/usr/local/libiconv \
--with-bz2 \
--with-curl \
--with-freetype-dir \
--with-gd \
--with-gettext \
--with-gmp \
--with-jpeg-dir \
--with-libmbfl \
--with-libxml-dir=/usr/ \
--with-mhash \
--with-mysqli=mysqlnd \
--with-mysql-sock \
--with-onig \
--with-openssl \
--with-openssl-dir \
--with-pcre-dir \
--with-pcre-jit \
--with-pcre-regex \
--with-pdo-mysql=mysqlnd \
--with-pdo-sqlite \
--with-pear \
--with-png-dir \
--with-readline \
--with-sqlite3 \
--with-xmlrpc \
--with-xsl \
--with-zlib \
--with-zlib-dir \
--enable-bcmath \
--enable-calendar \
--enable-exif \
--enable-fpm \
--enable-ftp \
--enable-gd-jis-conv \
--enable-maintainer-zts \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-shared  \
--enable-shmop \
--enable-sigchild \
--enable-soap \
--enable-static \
--enable-sockets  \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--enable-zip \
--disable-rpath \
--disable-ipv6

make && make install
```
安装完后输出如下，这里面提供了很多信息
```
Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20170718/
Installing PHP CLI binary:        /usr/local/php/bin/
Installing PHP CLI man page:      /usr/local/php/php/man/man1/
Installing PHP FPM binary:        /usr/local/php/sbin/
Installing PHP FPM defconfig:     /usr/local/php/etc/
Installing PHP FPM man page:      /usr/local/php/php/man/man8/
Installing PHP FPM status page:   /usr/local/php/php/php/fpm/
Installing phpdbg binary:         /usr/local/php/bin/
Installing phpdbg man page:       /usr/local/php/php/man/man1/
Installing PHP CGI binary:        /usr/local/php/bin/
Installing PHP CGI man page:      /usr/local/php/php/man/man1/
Installing build environment:     /usr/local/php/lib/php/build/
Installing header files:          /usr/local/php/include/php/
Installing helper programs:       /usr/local/php/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php/lib/php/
[PEAR] Archive_Tar    - installed: 1.4.3
[PEAR] Console_Getopt - installed: 1.4.1
[PEAR] Structures_Graph- installed: 1.1.1
[PEAR] XML_Util       - installed: 1.4.2
[PEAR] PEAR           - installed: 1.10.5
Wrote PEAR system config file at: /usr/local/php/etc/pear.conf
You may want to add: /usr/local/php/lib/php to your php.ini include_path
/root/php-7.2.3/build/shtool install -c ext/phar/phar.phar /usr/local/php/bin
ln -s -f phar.phar /usr/local/php/bin/phar
Installing PDO headers:           /usr/local/php/include/php/ext/pdo/
```

### 配置文件  
cp php.ini-production /usr/local/php/etc/php.ini  
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf  
vi /usr/local/php/etc/php.ini  
查找 cgi.fix_pathinfo= 并将其修改为如下：  
cgi.fix_pathinfo = 0  
查找 date.timezone 修改为：  
date.timezone = Asia/Shanghai  

vi /usr/local/php/etc/php-fpm.conf
```
[global]
pid = /usr/local/php/var/run/php-fpm.pid
error_log = /data/wwwlogs/php-fpm-error.log
log_level = notice

[www]
listen = /tmp/php-cgi.sock
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = dynamic
pm.max_children = 100
pm.start_servers = 20
pm.min_spare_servers = 20
pm.max_spare_servers = 30

request_terminate_timeout = 6000
#request_terminate_timeout = 3000
request_slowlog_timeout = 15s
slowlog = /data/wwwlogs/php-slow.log
```


## 安装扩展

### pcntl扩展pcntl.so

*如果前面--enable-pcntl就不需要这一步了*

在安装包里面就有

```
cd ./ext/pcntl
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

安装完成后已经告诉你模块安装到了  
/usr/local/php/lib/php/extensions/no-debug-zts-20131226/

```
Build complete.
Don't forget to run 'make test'.

Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-zts-20131226/
```

**其他模块以此类推，安装包有的扩展模块就不需要下载了，没有的需要下载，例如redis**  

### redis扩展redis.so
```
git clone https://github.com/phpredis/phpredis.git
cd phpredis
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

### mongo扩展mongo.so
```
wget http://pecl.php.net/get/mongo-1.6.16.tgz
tar -xf mongo-1.6.16.tgz && cd mongo-1.6.16
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```
*mongo.so拓展php7不支持，得使用mongodb.so拓展*

### mongodb扩展mongodb.so
```
wget http://pecl.php.net/get/mongodb-1.4.2.tgz
tar -xf mongodb-1.4.2.tgz && cd mongodb-1.4.2
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

### memcache扩展memcache.so
```
yum install libmemcached libmemcached-devel
git clone https://github.com/php-memcached-dev/php-memcached
cd php-memcached
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

### pgsql扩展pgsql.so
```
yum -y install postgresql-devel
cd ext/pgsql/
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

### pdo_pgsql扩展pdo_pgsql.so
```
yum -y install postgresql-devel
cd ext/pdo_pgsql/
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

### 将扩展写入配置文件  
vi /usr/local/php/php/php.ini  
添加
```
[opcache]
zend_extension="/usr/local/php/lib/php/extensions/no-debug-zts-20170718/opcache.so"
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=120
opcache.fast_shutdown=1
opcache.enable_cli=1
opcache.validate_timestamps=0

extension_dir = "/usr/local/php/lib/php/extensions/no-debug-zts-20170718/"
extension=redis.so
extension=memcached.so
extension=mongodb.so
extension=pgsql.so
extension=pdo_pgsql.so
```

### 配置 php-pfm 服务
```
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig php-fpm on
```
启动 php-fpm：  
service php-fpm start  
然后在配置 Nginx 后端支持的时候：  
```
location ~ \.php$ {
    try_files $uri =404;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    include fastcgi_params;
    fastcgi_pass unix:/tmp/php-cgi.sock;
}
```

## 其他
查看编译配置  
/usr/local/php/bin/php -i|grep configure  
查看已加载模块  
/usr/local/php/bin/php -m  
查看版本  
/usr/local/php/bin/php -v  
查看当前配置文件  
/usr/local/php/bin/php --ini  

## 参考
[Linux下安装libiconv使php支持iconv函数](http://www.osyunwei.com/archives/9195.html)  
[CentOS 6.5编译安装PHP5.6](https://blog.kuoruan.com/101.html)  
[编译安装php时候的参数说明](http://www.voidcn.com/article/p-zlhpkuok-ee.html)  
[PHP 编译安装 PHP各参数配置详解](http://www.jianshu.com/p/0a79847c8151)  
[编译安装redis，memcache，mongodb扩展](http://www.webyang.net/Html/web/article_288.html)  
[Linux下PHP安装Redis扩展和mongodb扩展](http://cpper.info/2016/06/03/Install-Redis-MongoDB-Plugin-of-PHP.html)  
[php扩展包官网](https://pecl.php.net/packages.php)  
[centos7下编译安装libiconv-1.14 error: ‘gets’ undeclared here (not in a function)](http://www.rootop.org/pages/3532.html)  
[CentOs7 安装php7和postgresql9.6安装及php pgsql扩展模块](https://www.datayang.com/article/12)  
[error: Please reinstall the libzip distribution](http://www.cnblogs.com/dongbo/p/7863115.html)  
[PHP7 下安装 memcache 和 memcached 扩展](http://www.lnmp.cn/install-memcache-and-memcached-extends-under-php7.html)  