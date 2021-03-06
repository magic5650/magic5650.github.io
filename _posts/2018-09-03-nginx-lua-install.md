---
layout: post
title: nginx lua 编译安装
author: magic
date:   2018-09-03
categories: Linux
tags: linux lua
permalink: /archivers/nginx-lua-install
---
* 目录
{:toc}

## 编译安装脚本
<!--more-->
> nginx-lua-build.sh

```
#!/bin/bash
#Date:2018-09-03
#Written by magic5650.
#Readme:install nginx + lua.

#step 0
mkdir /usr/local/nginx
mkdir nginx_lua_build

#step 1
wget -q http://nginx.org/download/nginx-1.13.6.tar.gz
tar xf nginx-1.13.6.tar.gz

#step 2
wget -q https://www.openssl.org/source/openssl-1.1.0i.tar.gz
tar xf openssl-1.1.0i.tar.gz

wget -q https://sourceforge.mirrorservice.org/p/pc/pcre/pcre/8.38/pcre-8.38.tar.gz
tar xf pcre-8.38.tar.gz

wget -q https://nchc.dl.sourceforge.net/project/libpng/zlib/1.2.11/zlib-1.2.11.tar.gz
tar xf zlib-1.2.11.tar.gz

#step 3
wget -q http://luajit.org/download/LuaJIT-2.1.0-beta3.zip
unzip LuaJIT-2.1.0-beta3.zip >> /dev/null
cd LuaJIT-2.1.0-beta3
#modify Makefile
sed -i '/^export PREFIX=/s/\/usr\/local/\/usr\/local\/nginx\/luajit-2.1.0/' Makefile
make
make install PREFIX=/usr/local/nginx/luajit-2.1.0
ln -sf /usr/local/nginx/luajit-2.1.0/bin/luajit-2.1.0-beta3 /usr/local/nginx/luajit-2.1.0/bin/luajit
/usr/local/nginx/luajit-2.1.0/bin/luajit -e "print(package.cpath)" | sed -n 's/;/;\n/gp'
cd -

#step 4
wget -q https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz
tar xf v0.3.1rc1.tar.gz

git clone https://github.com/openresty/lua-nginx-module.git
mv lua-nginx-module lua-nginx-module-0.10.14

wget -q https://github.com/openresty/stream-lua-nginx-module/archive/v0.0.5.tar.gz
tar xf v0.0.5.tar.gz

git clone https://github.com/wdaike/ngx_upstream_jdomain.git

#step 5
cd nginx-1.13.6
export LUAJIT_LIB=/usr/local/nginx/luajit-2.1.0/lib
export LUAJIT_INC=/usr/local/nginx/luajit-2.1.0/include/luajit-2.1

./configure \
--prefix=/usr/local/nginx \
--with-file-aio \
--with-http_v2_module \
--with-http_dav_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_sub_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_ssl_module \
--with-openssl=../openssl-1.1.0i \
--with-zlib=../zlib-1.2.11 \
--with-pcre=../pcre-8.38 \
--with-pcre-jit \
--with-stream \
--with-stream_ssl_module \
--with-ld-opt=-Wl,-rpath,/usr/local/nginx/luajit-2.1.0/lib \
--add-module=../ngx_devel_kit-0.3.1rc1 \
--add-module=../lua-nginx-module-0.10.14 \
--add-module=../stream-lua-nginx-module-0.0.5 \
--add-module=../ngx_upstream_jdomain

make -j2
make install

#step 6
cd -
git clone https://github.com/openresty/lua-cjson.git
cd lua-cjson
make
make install LUA_CMODULE_DIR=/usr/local/nginx/luajit-2.1.0/lib/lua/5.1 LUA_INCLUDE_DIR=/usr/local/nginx/luajit-2.1.0/include/luajit-2.1
ls -l /usr/local/nginx/luajit-2.1.0/lib/lua/5.1/cjson.so
cd -


#step 7
git clone https://github.com/openresty/lua-resty-core
git clone https://github.com/openresty/lua-resty-lrucache
git clone https://github.com/openresty/lua-resty-limit-traffic

mkdir /usr/local/nginx/lualib/
cd lua-resty-core/lib
cp -r . /usr/local/nginx/lualib/
tree -L 2 /usr/local/nginx/lualib
cd -
cd lua-resty-lrucache/lib
cp -r . /usr/local/nginx/lualib/
tree -L 2 /usr/local/nginx/lualib
cd -
cd lua-resty-limit-traffic/lib
cp -r . /usr/local/nginx/lualib/
tree -L 2 /usr/local/nginx/lualib
cd -


#at last
cd /usr/local/nginx
tree -L 2 
/usr/local/nginx/sbin/nginx -V
```

查看
> /usr/local/nginx/sbin/nginx -V

```
nginx version: nginx/1.13.6
built by gcc 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10) 
built with OpenSSL 1.1.0i  14 Aug 2018
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-file-aio --with-http_v2_module --with-http_dav_module --with-http_realip_module --with-http_flv_module --with-http_mp4_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --with-http_ssl_module --with-openssl=../openssl-1.1.0i --with-zlib=../zlib-1.2.11 --with-pcre=../pcre-8.38 --with-pcre-jit --with-stream --with-stream_ssl_module --with-ld-opt=-Wl,-rpath,/usr/local/nginx/luajit-2.1.0/lib --add-module=../ngx_devel_kit-0.3.1rc1 --add-module=../lua-nginx-module-0.10.14 --add-module=../stream-lua-nginx-module-0.0.5 --add-module=../ngx_upstream_jdomain
```

## 主配置文件
> nginx.conf

```
user www-data;
worker_processes 4;
worker_rlimit_nofile 262140;
error_log  logs/error.log warn;
pid  run/nginx.pid; #需跟nginx.service里面定义一致

events 
{
	use epoll;
	worker_connections  65535;
}


http 
{
	include mime.types;
	default_type application/octet-stream;
	# Update charset_types due to updated mime.types
	charset_types text/xml text/plain text/vnd.wap.wml application/x-javascript application/rss+xml text/css application/javascript application/json;
	
	sendfile on;
	aio on;
	directio 512;
	output_buffers 1 128k;
	log_not_found off;
	keepalive_timeout  60;
	server_tokens off;
	
	# Compression
	gzip on;

	# Setup gzip http version 1.0 
	gzip_http_version 1.0;

	# Disable gzip compression for IE6, because IE6 supports gzip not friendly.
	gzip_disable     "MSIE [1-6]\.(?!.*SV1)";

	# Compression level (1-9).
	# 5 is a perfect compromise between size and cpu usage, offering about
	# 75% reduction for most ascii files (almost identical to level 9).
	gzip_comp_level 5;

	# Don’t compress anything that’s already small and unlikely to shrink much
	# if at all (the default is 20 bytes, which is bad as that usually leads to
	# larger files after gzipping).
	gzip_min_length  1k;
	gzip_buffers     4 8k;

	# Compress data even for clients that are connecting to us via proxies
	# identified by the “Via” header (required for CloudFront).
	gzip_proxied any;

	# Tell proxies to cache both the gzipped and regular version of a resource
	# whenever the client’s Accept-Encoding capabilities header varies;
	# Avoids the issue where a non-gzip capable client (which is extremely rare
	# today) would display gibberish if their proxy gave them the gzipped version.
	gzip_vary on;

	# Compress all output labeled with one of the following MIME-types.
	gzip_types application/atom+xml application/javascript application/json application/rss+xml application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/svg+xml image/x-icon text/css text/plain text/x-component;
	# text/html is always compressed by HttpGzipModule
	
	log_format main '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$request_time" "$upstream_response_time" "$http_x_forwarded_for"';

	access_log logs/${server_name}.access.log main;

	fastcgi_intercept_errors on;
	error_page   500 502 503 504  /50x.html;

	server_names_hash_max_size 4096;
	
	geo $allow {
		default	0;
		
		127.0.0.1 1;
		10.0.0.0/8 1;
		192.168.0.0/16 1;
		172.16.0.0/12 1;
	}

	server 
	{
		listen 80 default_server;
		server_name  _;
		access_log off;
		
		# use for health check
		location / 
		{
			if ($allow = 1) {
				return 200;
			}
			if ($allow = 0) {
				return 403;
			}
		}
		# use for monitor
		location /nginx_status
		{
			stub_status on;
			access_log off;
			allow 127.0.0.1;
			deny all;
		}
	}

	server 
	{
		listen 8080 default_server;
		server_name  _;
		access_log off;
		
		# use for health check
		location / 
		{
			if ($allow = 1) {
				return 200;
			}
			if ($allow = 0) {
				return 403;
			}
		}
		# use for monitor
		location /nginx_status
		{
			stub_status on;
			access_log off;
			allow 127.0.0.1;
			deny all;
		}
	}

	# set search paths for pure Lua external libraries (';;' is the default path):
	lua_package_path "/usr/local/nginx/lualib/?.lua;;";
	# set search paths for Lua external libraries written in C (can also use ';;');
	lua_package_cpath '/usr/local/nginx/luajit-2.1.0/lib/lua/5.1/?.so;;';

	lua_shared_dict limit_req_store 100m;
	lua_shared_dict cc_req_store 100m;
	
	include /data/services/nginx_vhost/*.conf;
}
```

## systemd启动配置
> nginx.service

```
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecStartPost=/bin/sleep 0.1
ExecReload=/usr/local/nginx/sbin/nginx -s reload
#ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
ExecStopPost=
Restart=on-failure
#RestartSec=1
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## 启动脚本
低版本的linux使用，需要安装 start-stop-daemon
> nginx.sh

```
#! /bin/sh

### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/local/nginx/sbin/nginx
LOCK_FILE="/usr/local/nginx/run/auto_restart_when_stop"
NAME=nginx
DESC=nginx

test -x $DAEMON || exit 255

set -e

. /lib/lsb/init-functions

test_nginx_config() {
  if $DAEMON -t $DAEMON_OPTS
  then
    return 0
  else
    return $?
  fi
}

case "$1" in
  start)
        echo -n "Starting $DESC: "
        test_nginx_config
        start-stop-daemon --start --quiet --pidfile /usr/local/nginx/run/$NAME.pid \
                --exec $DAEMON -- $DAEMON_OPTS || true
        echo "$NAME."
		touch $LOCK_FILE
        ;;
  stop)
        echo -n "Stopping $DESC: "
        start-stop-daemon --stop --quiet --pidfile /usr/local/nginx/run/$NAME.pid \
                --exec $DAEMON || true
        echo "$NAME."
		rm -f $LOCK_FILE
        ;;
  restart|force-reload)
        echo -n "Restarting $DESC: "
        start-stop-daemon --stop --quiet --pidfile \
                /usr/local/nginx/run/$NAME.pid --exec $DAEMON || true
        sleep 1
        test_nginx_config
        start-stop-daemon --start --quiet --pidfile \
                /usr/local/nginx/run/$NAME.pid --exec $DAEMON -- $DAEMON_OPTS || true
		touch $LOCK_FILE
        echo "$NAME."
        ;;
  reload)
        echo -n "Reloading $DESC configuration: "
        test_nginx_config
        start-stop-daemon --stop --signal HUP --quiet --pidfile /usr/local/nginx/run/$NAME.pid \
            --exec $DAEMON || true
        echo "$NAME."
        ;;
  configtest)
        echo -n "Testing $DESC configuration: "
        if test_nginx_config
        then
          echo "$NAME."
        else
          exit $?
        fi
        ;;
  status)
        status_of_proc -p /usr/local/nginx/run/$NAME.pid "$DAEMON" nginx && exit 0 || exit $?
        ;;
  *)
        echo "Usage: $NAME {start|stop|restart|reload|force-reload|status|configtest}" >&2
        exit 1
        ;;
esac

exit 0

```




