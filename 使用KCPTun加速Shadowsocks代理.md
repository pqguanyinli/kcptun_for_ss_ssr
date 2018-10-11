## 使用KCPTun加速Shadowsocks代理

### 前言

在国内如果经常要访问一些诸如Google、Facebook、Youtube等根本不存在的网站，最便捷的方式就是用Shadowsocks搭梯子。但是往往梯子太长，即使梯子带宽足够宽，线路质量也是不忍直视。本文介绍如何使用KCPTun来加速Shadowsocks代理速度。

> **本文不对Shadowsocks的设置进行说明，默认Shadowsocks服务端与客户端已部署且浏览器代理服务器已经正确设置。** 

### KCPTun介绍

KCPTun是一个使用可信UDP来加速TCP传输速度的网络软件。 

官网：https://github.com/xtaci/kcptun

我们知道TCP协议是可信的数据流传输协议。简单来说，如果数据传输过程中发生了丢包，TCP协议会重新发送相应数据包，如果数据包到达顺序与发送顺序不一致，TCP协议会进行数据包重组，即：TCP协议可以通过控制帧来保证数据流的传输顺序和正确性。但TCP协议的控制机制比较复杂，在线路质量差导致丢包率极高时，传输效率就会指数级下降。

UDP协议是数据报协议，由于比TCP简单得多，传输效率和延迟率都要优于TCP协议，但UDP协议不是可信传输协议，不能保证数据正确与可达，所以只能应用在一些对单个数据包的正确与可达不是要求很严格（比如：IM）、但对数据传输延迟率有很高要求（比如：视频流或者多人在线游戏）的场合。

那么，有没有一种传输协议既可信又能保证传输效率呢？这就是R-UDP：可信UDP协议，一种在UDP协议基础上增加了部分TCP的控制逻辑来保证数据正确完整的协议。目前已经有很多可用库（rUDP、enet等等），虽然没有成为像TCP/UDP这样的标准协议，但已经广泛运用到了大型多人在线游戏等领域。KCPTun就是利用这种可信UDP（KCP over UDP），来大幅加速TCP传输效率的。

> **KCP 是一个快速可靠协议，能以比 TCP浪费10%-20%的带宽的代价，换取平均延迟降低 30%-40%，且最大延迟降低三倍的传输效果。 
参见：http://blog.csdn.net/linshuhe1/article/details/52191625**

KCPTun原理图： 

![][img1]

KCPTun官方性能实测值：从68Kbps提高到31Mbps，飞一般的感受！ 

![][img2]

### 梯子服务端

#### 解压KCPTun

1) 创建KCPTun文件存放路径
```
# mkdir ~/kcptun
```
2) 解压kcptun-linux-amd64-20170315.tar.gz
```
# tar xfvz ./kcptun-linux-amd64-20170315.tar.gz -C ~/kcptun

# pwd
/foo/kcptun
# ls -l
total 7032
-rwxr-xr-x 1 foo bar 3614080 Mar 10 01:50 client_linux_amd64
-rwxr-xr-x 1 foo bar 3581152 Mar 10 01:50 server_linux_amd64
```
#### 启动kcptun服务端
```
# /foo/kcptun/server_linux_amd64 -t 127.0.0.1:9000 -l :29900 --key mykey --crypt aes-128 --mode fast2
```
> **这样启动的server_linux_amd64只能在前台运行，终端关闭或者Ctrl+c之后kcptun服务程序就关闭了，所以还需要进行下一步。**

参数说明：
```
-t 127.0.0.1:9000  指向本机Shadowsocks的服务端口，本例中是9000
-l :29900          KCPTun服务端端口，这个值在下一节KCPTunGUI设置中需要用到
--key mykey        KCPTun服务端与客户端连接用的secret key
--crypt aes-128    KCPTun服务端与客户端通信时使用aes-128加密（不必须，基于安全性的考虑推荐使用）
--mode fast2       KCPTun服务端与客户端数据传输模式。
根据官方文档：
   响应速度：  fast3 > fast2 > [fast] > normal > default
   有效载荷比：default > normal > [fast] > fast2 > fast3
```
#### 设置开机启动

如果想要机器每次重启之后自动运行kcptun的话，就需要添加开机启动脚本。
```
# sudo touch /etc/init.d/kcptun
# sudo chmod +x /etc/init.d/kcptun
# sudo vim /etc/init.d/kcptun
```
在/etc/init.d/kcptun文件中写入以下内容：
```
#!/bin/bash

#
# KCPTun       Startup script for the KCPTun Server
#
# chkconfig: - 90 10
# 
# description: The KCPTun is a secure tunnel based On KCP with N:M Multiplexing. \
#              (https://github.com/xtaci/kcptun)
# processname: kcptun
# config: /etc/sysconfig/kcptun
# pidfile: /var/run/kcptun.pid
# 

### BEGIN INIT INFO
# Provides: KCPTun
# Required-Start: $network $syslog $local_fs $remote_fs $named
# Required-Stop: $network $local_fs $remote_fs 
# Should-Start: 
# Should-Stop:        
# Default-Start:      
# Default-Stop:  
# Short-Description: Start and stop KCPTun Server
# Description: The KCPTun is a secure tunnel based On KCP with N:M Multiplexing.
### END INIT INFO


# Author: farawayzheng <http://blog.csdn.net/farawayzheng_necas>
# 
# To install:
#   Copy this file to /etc/rc.d/init.d/kcptun
#   $ chkconfig --add kcptun
#
#
# To uninstall:
#   $ chkconfig --del kcptun
#   

#export PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/opt/bin:

BASE=$(basename $0)

# modify these in /etc/sysconfig/$BASE (/etc/sysconfig/kcptun)
KCPTUN=/foo/kcptun/server_linux_amd64

KCPTUN_PIDFILE=/var/run/$BASE.pid
KCPTUN_LOGFILE=/var/log/$BASE.log
KCPTUN_LOCKFILE=/var/lock/subsys/$BASE
KCPTUN_OPTS="-t 127.0.0.1:9000 -l :29900 --key mykey --crypt aes-128 --mode fast2"
KCPTUN_DESC="KCPTUN"

# Source function library.
. /etc/rc.d/init.d/functions

if [ -f /etc/sysconfig/$BASE ]; then
    . /etc/sysconfig/$BASE
fi

# Check kcptun server is present
if [ ! -x $KCPTUN ]; then
    echo "$KCPTUN not present or not executable!"
    exit 1
fi

RETVAL=0
STOP_TIMEOUT=${STOP_TIMEOUT-10}

start() {

    if [ -f ${KCPTUN_LOCKFILE} ]; then

        if [ -s ${KCPTUN_PIDFILE} ]; then
            echo "$BASE might be still running, stop it first!"
            killproc -p ${KCPTUN_PIDFILE} -d ${STOP_TIMEOUT} $KCPTUN
        else
            echo "$BASE was not shut down correctly!"
        fi

        rm -f ${KCPTUN_PIDFILE} ${KCPTUN_LOCKFILE}
        sleep 2
    fi

    echo -n $"Starting $BASE: "
    $KCPTUN --log ${KCPTUN_LOGFILE} $KCPTUN_OPTS &
    RETVAL=$?

    if [ "$RETVAL" = "0" ]; then
        success
        sleep 2
        ps -A o pid,cmd | grep "$KCPTUN --log ${KCPTUN_LOGFILE} $KCPTUN_OPTS" | awk '{print $1}' | head -n 1 > ${KCPTUN_PIDFILE} 
    else
        failure
    fi
    echo

    [ $RETVAL = 0 ] && touch ${KCPTUN_LOCKFILE}
    return $RETVAL
}

stop() {
    echo -n $"Stopping $BASE: "
    killproc -p ${KCPTUN_PIDFILE} -d ${STOP_TIMEOUT} $KCPTUN
    RETVAL=$?
    echo
    [ $RETVAL = 0 ] && rm -f ${KCPTUN_PIDFILE} ${KCPTUN_LOCKFILE}
    return $RETVAL
}


case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  status)
        status -p ${KCPTUN_PIDFILE} $KCPTUN
        RETVAL=$?
        ;;
  restart)
        stop
        start
        ;;
  *)
        echo $"Usage: $BASE { start | stop | restart | status }"
        RETVAL=2
        ;;
esac

exit $RETVAL
```


> **脚本中的 KCPTUN_OPTS=”-t 127.0.0.1:9000 -l :29900 –key mykey –crypt aes-128 –mode fast2” 可以根据具体情况来进行修改**

运行以下命令激活开机启动：
```
# chkconfig --add kcptun
# chkconfig kcptun on
```
启动kcptun：
```
# sudo service kcptun start
```
查看kcptun进程是否启动了：
```
# sudo ps aux | grep server_linux_amd64
root     17007  0.0  1.2  26244  6736 ?        Sl   Mar14   1:54 /foo/kcptun/server_linux_amd64 --log /var/log/kcptun.log -t 127.0.0.1:9000 -l :29900 --key mykey --crypt aes-128 --mode fast2
```
如果有像上面信息说明kcptun进程正常启动。 

### 梯子客户端

首先将下载后的压缩文件解压至某目录，比如可以把kcptun-windows-amd64-20170315.tar.gz和kcptun_gclientv.1.1.1.zip都解压到c:\kcptun目录下。

#### 设置KCPTunGUI：

点击kcptun_gclient.exe启动KCPTun客户端GUI。

![][img3]

> **Shadowsocks的公网地址就是Shadowsocks服务端安装主机的公网IP地址，在主机服务商（AWS等等）的页面上可以查到，比如：**

![][img4]

> **注意：是主机公网IP地址，不是主机私网IP！**

#### 设置Shadowsocks GUI：

![][img5]

1. 服务器IP设置成：127.0.0.1 

2. 服务端口号设置成：KCPTunGUI设置中的本地侦听端口（本例中9527） 

3. 点击确认按钮 

至此，KCPTun设置全部完成，打开浏览器测试一下效果吧。 

> **KCPTun官方下载地址：https://github.com/xtaci/kcptun/releases/**
>
> **Windows版KCPTun客户端GUI：**
> **https://github.com/dfdragon/kcptun_gclient/releases/**
>
> **Mac版KCPTun客户端GUI：**
> **https://github.com/dfdragon/kcptun_xclient/releases/**
>
> **Shadowsocks windows客户端: https://github.com/shadowsocks/shadowsocks-windows/releases/**
>
> **Shadowsocks 安卓客户端: https://github.com/shadowsocks/shadowsocks-android/releases/**
>
> **kcptun 安卓Shadowsocks客户端插件: https://github.com/shadowsocks/kcptun-android/releases**

[img1]:/images/1.png
[img2]:/images/2.png
[img3]:/images/3.png
[img4]:/images/4.png
[img5]:/images/5.png
