---
layout: post
title: tomcat编译打包
author: magic
date:   2016-10-11
categories: Linux
tags: linux
permalink: /archivers/tomcat-build
---
* 目录
{:toc}

## tomcat编译打包

### 编译环境

    ubuntu 9  Linux ubuntu 2.6.28-11-server 
    sudo strings /lib/libc.so.6 |grep GLIBC
    ...
    GLIBC_2.9
    目录:/usr/loca/src

###  下载包系统上的最新的tomcat版本
 
### 1.下载、安装apr
```
cd /usr/loca/src
sudo  wget http://apache.fayea.com//apr/apr-1.5.2.tar.gz
sudo tar -xzf apr-1.5.2.tar.gz
cd apr-1.5.2
./configure --prefix=/usr/local/apr
sudo make
sudo make install
```
### 2.下载、安装apr-util
```
cd /usr/local/src
sudo wget http://mirrors.hust.edu.cn/apache//apr/apr-util-1.5.4.tar.gz
sudo tar -xzf apr-util-1.5.4.tar.gz
cd apr-util-1.5.4
./configure --with-apr=/usr/local/apr
sudo make 
sudo make install
```
### 3.下载、安装openssl
```
cd /usr/local/src
sudo wget https://www.openssl.org/source/openssl-1.0.2k.tar.gz
sudo tar -zxf openssl-1.0.2k.tar.gz
cd openssl-1.0.2k
./config --prefix=/usr/local/openssl -fPIC
sudo make
sudo make install
/usr/local/openssl/bin/openssl version
(OpenSSL 1.0.2k  26 Jan 2017)
```
**确认版本信息是1.0.2k**
### 4.下载、安装tomcat7
```
cd /usr/local/src
sudo wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-7/v7.0.75/bin/apache-tomcat-7.0.75.tar.gz
sudo tar -zxf apache-tomcat-7.0.75.tar.gz
export CATALINA_HOME=/usr/local/src/apache-tomcat-7.0.75
cd apache-tomcat-7.0.75/bin
sudo tar -xzf tomcat-native.tar.gz
cd tomcat-native-1.2.10-src
./configure --with-apr=/usr/local/apr \
--with-ssl=/usr/local/openssl \
--with-java-home=$JAVA_HOME \
--prefix=$CATALINA_HOME
sudo make
sudo make install
```

安装完后你将会发现在 **$CATALINA_HOME/lib** 目录里多出 **libtcnative** 之类的库
```
ls $CATALINA_HOME/lib|grep libtcnative*
libtcnative-1.a
libtcnative-1.la
libtcnative-1.so
libtcnative-1.so.0
libtcnative-1.so.0.2.10
```

### 5.将aprlib全部复制到$CATALINA_HOME/lib
```
cd /usr/local/apr/lib
sudo cp -r * $CATALINA_HOME/lib
```
**$CATALINA_HOME/bin** 目录里编辑（没有的话则新建）**setenv.sh** 文件（这个文件用于添加 Tomcat 启动参数），增加如下两行：

    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CATALINA_HOME/lib
    export LD_LIBRARY_PATH

### 6.复制scripts和setenv.sh

**将老版本的tomcat目录下的scripts文件夹全部复制到新版本**

**将老版本的tomcat的bin目录下面的setenv.sh复制过来**
    
setenv.sh
```
LOG_BASE="/data/weblog/tomcat"
JAVA_HOME=/usr/local/java;
JRE_HOME=/usr/local/java/jre;
CATALINA_HOME=/usr/local/tomcat;
TOMCAT_USER=www-data
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CATALINA_HOME/lib;
export LD_LIBRARY_PATH;

DNS=$(echo $CATALINA_BASE|awk -F'/' '{print $(NF)}')
LOG_DIR="$LOG_BASE/$DNS"
[ ! -d $LOG_DIR ] && mkdir -p $LOG_DIR
if [ ! -e $CATALINA_BASE/logs ]
then
        ln -s $LOG_DIR $CATALINA_BASE/logs
elif [ "$(readlink $CATALINA_BASE/logs)" != "$LOG_DIR" ]
then
        mv $CATALINA_BASE/logs $CATALINA_BASE/logs.bak.$(date +%F)
        ln -s $LOG_DIR $CATALINA_BASE/logs
fi

chown -R $TOMCAT_USER:$TOMCAT_USER $LOG_DIR $CATALINA_BASE/logs
CATALINA_OUT=$CATALINA_BASE/logs/catalina.log
CATALINA_PID=$CATALINA_BASE/run/tomcat.pid
JSVC_OPTS='-jvm server';
JAVA_OPTS="-Xms1024m -Xmx2048m -Duser.timezone=Asia/Shanghai -Xmn512m -XX:PermSize=192m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Duser.timezone=Asia/Shanghai -Dfile.encoding=UTF-8 "
```

### 7.以 Daemon 方式运行 Tomcat

```
cd /usr/local/src
cd apache-tomcat-7.0.75/bin
sudo tar -zxf commons-daemon-native.tar.gz
cd commons-daemon-1.0.15-native-src/unix
sudo ./configure --with-java=$JAVA_HOME
sudo make
#你会发现当前目录产生一个名字为  jsvc 的可执行文件，把这个文件复制到 $CATALINA_HOME/bin 目录即可。
sudo cp jsvc ../..
```

至此新版tomcat包打包完成

公司服务器有tomcat启动脚本 **/etc/init.d/tomcat**  (以daemon方式启动)

```
#!/bin/bash
JVMAGENT="jvm_agent_aHR0cDovL3dxLnN5c29wLmR1b3dhbi5jb20v.war"
JVMAGENT_DIR="/data/webapps/jvmagent"

if [ $# -ne 2 ]  && [ $# -ne 1 ]
then
	echo "Usage: /etc/init.d/tomcat (all|xxx.yy.com)  (start|stop|restart)"
    exit 1
fi

if [ $# -eq 1 ] || [ "$1" = "all" ]
then
	for WEBAPPSDNS in $(/bin/ls /data/webapps/|grep -vE "bbb.yy.com|dragon|jvmagent")
	do
		if [ -d /data/webapps/${WEBAPPSDNS} -a "${WEBAPPSDNS}" != "" ];then
			[ -f ${JVMAGENT_DIR}/${JVMAGENT} ] || (mkdir -p ${JVMAGENT_DIR} && cd ${JVMAGENT_DIR} && wget http://wq.sysop.duowan.com/${JVMAGENT} > /dev/null 2>&1)
			[ -L /data/webapps/${WEBAPPSDNS}/.${JVMAGENT} ] || ln -s ${JVMAGENT_DIR}/${JVMAGENT} /data/webapps/${WEBAPPSDNS}/.${JVMAGENT}
		fi
	done

	[ $# -eq 1 ] && ACTION=$1 || ACTION=$2
	for DNS in $(/bin/ls -d /data/services/tomcat_base/*/ |awk -F'/' '{print $(NF-1)}')
	do
		$0 $DNS $ACTION
	done
	exit 0
fi

CATALINA_HOME=/usr/local/tomcat
CATALINA_BASE=/data/services/tomcat_base/$1
PID_FILE="$CATALINA_BASE/run/tomcat.pid"
[[ $( sed -r '/[[:space:]]*<!--/,/-->[[:space:]]*$/d' $CATALINA_BASE/conf/server.xml | egrep '^[[:space:]]*<Connector.*port=.*protocol="HTTP/1.1"|^[[:space:]]*<Connector.*port=.*protocol="org.apache.coyote.http11.Http11NioProtocol"' ) =~ (.*)port=\"(.*)\"\ protocol=(.*) ]]
PORT="${BASH_REMATCH[2]}"
[[ ".$PORT" == "." ]] || [[ "$PORT" -le 8080 ]] ||[[ "$PORT" -gt 8100 ]] &&  { echo -e "\nPORT in  conf server.xml WRONG.\n";exit 3; }

case "$2" in
start)
	if [ ! -f $PID_FILE ] || ! ps -p $(cat $PID_FILE) >/dev/null 2>&1 && ! netstat -tln|awk '{print $4}'|cut -d: -f 2|grep -wq $PORT
	then
    	/usr/local/tomcat/bin/daemon.sh --catalina-base  $CATALINA_BASE start
		STATUS=$?
		sleep 5
		netstat -tln|awk '{print $4}'|cut -d: -f 2|grep -wq $PORT && STATUS=0
		[ $STATUS -eq 0 ] && touch $CATALINA_BASE/run/auto_restart_when_stop
		if [ "$1" != "bbb.yy.com" -o "$1" != "dragon" -o "$1" != "jvmagent" ];then
			[ -f ${JVMAGENT_DIR}/${JVMAGENT} ] || (mkdir -p ${JVMAGENT_DIR} && cd ${JVMAGENT_DIR} && wget http://wq.sysop.duowan.com/${JVMAGENT} > /dev/null 2>&1)
			[ -L /data/webapps/$1/.${JVMAGENT} ] || ln -s ${JVMAGENT_DIR}/${JVMAGENT} /data/webapps/$1/.${JVMAGENT}
		fi
	else
		echo -en "\n Tomcat $1 already running!\n\n"
		STATUS=254
	fi
    ;;
stop)
    /usr/local/tomcat/bin/daemon.sh --catalina-base  $CATALINA_BASE stop
	sleep 5
	netstat -tln|awk '{print $4}'|cut -d: -f 2|grep -wq $PORT && kill $(ss -tlnp|grep  -w $PORT|awk -F',' '{print $(NF-1)}')
	sleep 3
	! netstat -tln|awk '{print $4}'|cut -d: -f 2|grep -wq $PORT  && /bin/rm -f $CATALINA_BASE/run/auto_restart_when_stop && STATUS=0 || STATUS=2
    ;;
force-stop)
	PID=$(cat $PID_FILE)
	kill -9 $PID $(ps --no-headers --format='ppid' $PID) &&	/bin/rm -f $CATALINA_BASE/run/auto_restart_when_stop 
	test $? -eq 0 && STATUS=0 || STATUS=3
	;;
restart)
	$0 $1 stop
	STATUS=$?
	if [ $STATUS -eq 0 ] || [ $STATUS -eq 255 ]
	then
		sleep 5
    	$0 $1 start
		STATUS=$?
	fi
    ;;
status)
	if [ ! -f $PID_FILE ] 
	then
		echo "tomcat pid file not exsit, $1 not running."
		exit 1
	elif [ ! ps -p $(cat $PID_FILE) >/dev/null 2>&1 ]
	then
		echo "tomcat pid file exsist, but process not exits, $1 not running."
		echo "removing tomcat pid file: $PID_FILE"
		rm -fv $PID_FILE
		exit 1
	else
		exit 0
	fi
	;;

*)
	echo "Usage: /etc/init.d/tomcat (all|xxx.yy.com) (start|stop|force-stop|restart|status)"
	exit 1
esac

case $STATUS in
0)
	echo "DONE."	
	;;
1)
	echo -en "\nFAILED TO START TOMCAT $1 ! STATUS= $STATUS \n\n"
	;;
2)
	echo -en "\nFAILED TO STOP TOMCAT $1 ! STATUS= $STATUS \n\n"
	;;
3)
	echo -en "\nFAILED TO FORCE STOP TOMCAT $1 ! STATUS= $STATUS \n\n"
	;;
esac

exit $STATUS

```

在包系统安装或升级tomcat
(安装不会删除老版本,删除老的tomcat版本后,会停止tomcat实例,所以卸载老版本后需要在包系统启动;
升级则会删除老版本并且重启实例)

现在重新启动 Tomcat 服务就会自动应用上 APR 连接器了，检验的方法也很简单，查看 Tomcat 的日志输出文件（位于 $CATALINA_HOME/logs）catalina.log，如果发现有如下标有红色字的行即表示 APR 已经应用成功：
```
sudo /etc/init.d/tomcat bbb.yy.com restart
cd /data/weblog/tomcat/bbb.yy.com
cat cat catalina.log|grep -C 5 apr

INFO: Deployment of web application directory /data/webapps/bbb.yy.com/ROOT has finished in 73 ms
Mar 10, 2017 12:19:56 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["http-apr-8081"]
Mar 10, 2017 12:19:56 PM org.apache.catalina.connector.MapperListener findDefaultHost
WARNING: Unknown default host [localhost] for connector [Connector[HTTP/1.1-8081]]
Mar 10, 2017 12:19:56 PM org.apache.coyote.AbstractProtocol start
INFO: Starting ProtocolHandler ["ajp-apr-8021"]
Mar 10, 2017 12:19:56 PM org.apache.catalina.connector.MapperListener findDefaultHost
WARNING: Unknown default host [localhost] for connector [Connector[AJP/1.3-8021]]
Mar 10, 2017 12:19:56 PM org.apache.catalina.startup.Catalina start
INFO: Server startup in 1798 ms
```

潜龙发布的tomcat项目,CATALINA_BASE是/data/services/tomcat_base/$DNS,里面也有setenv.sh
CATALINA_HOME是/usr/local/tomcat(这是个软链接,链接至最新的tomcat版本)