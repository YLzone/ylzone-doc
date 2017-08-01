========================
[部署]-集群部署zookeeper
========================


零、背景介绍
------------

0.1 部署说明::

    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    操作用户: root
    运行用户: zookep
    依赖说明: jdk1.8
    软件版本: zookeeper-3.4.9
    所需文件: zookeeper-3.4.9.tar.gz

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - zookeeper-server
  * - VM02
    - 192.168.182.102
    - zookeeper-server
  * - VM03
    - 192.168.182.103
    - zookeeper-server

0.3 目录说明::
    
    部署位置: /opt/zookeeper
    配置目录: /data/zookeeper/conf
    数据目录: /data/zookeeper/data
    日志目录: /data/zookeeper/logs
    PID 目录: /data/zookeeper/vars/run
    
0.4 端口介绍::

    2181: 表示和客户端通信的接口
    2888: 表示集群间交换信息的端口；
    3888: 表示集群选取"Ladaer"的通信端口。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进>入，蓝色为说明提示。



一、解决依赖
----------

1.1 安装依赖组件 ``所有节点``::

安装jdk::

    $ mkdir /usr/java
    $ tar xf jdk-8u60-linux-x64.gz -C /usr/java
    $ ln -sv /usr/java/jdk1.8.0_60 /usr/java/latest
    $ ln -sv /usr/java/latest /usr/java/default
    $ chown -R root:root /usr/java/jdk1.8.0_60
    $ echo 'export JAVA_HOME=/usr/java/default' > /etc/profile.d/java.sh
    $ echo 'export PATH=${PATH}:${JAVA_HOME}/bin' >> /etc/profile.d/java.sh
    $ source /etc/profile.d/java.sh

1.2 创建运行用户 ``所有节点``::

    $ useradd -s /sbin/nologin -u 2181 zookep

1.3 修改hosts文件 ``所有节点``:

.. code-block:: bash

    $ vim /etc/hosts
    # 添加如下内容
    192.168.182.101 VM01
    192.168.182.102 VM02
    192.168.182.103 VM03
    
.. note::

    如果你使用了内部DNS服务器则不需要修改此文件，否则请按照操作实行。

二、安装程序
----------

2.1 解压软件包 ``所有节点``::

    $ cd /tmp
    $ tar xf zookeeper-3.4.9.tar.gz -C /opt
    $ mv /opt/zookeeper-3.4.9 /opt/zookeeper
    $ echo "version: zookeeper-3.4.9" >> /opt/zookeeper/VERSION.md

2.2 整理程序目录 ``所有节点``::
    
    $ mv /opt/zookeeper/conf /opt/zookeeper/conf.orig
    $ rm -fv /opt/zookeeper/zookeeper-3.4.9.jar.{asc,md5,sha1}
    $ rm -fv /opt/zookeeper/bin/{README.txt,*.cmd}
    $ rm -rfv /opt/zookeeper/lib/{*.txt,cobertura,jdiff}
    $ rm -rfv /opt/zookeeper/{recipes,src,docs,contrib,dist-maven,*.txt,*.xml}

2.3 创建所需目录 ``所有节点``::

    $ mkdir -p /data/zookeeper/{conf,data,logs,vars}
    $ mkdir -p /data/zookeeper/vars/{run,tmp}

2.4 创建所需文件 ``所有节点``::

    $ cp /opt/zookeeper/conf.orig/* /data/zookeeper/conf
    $ touch /data/zookeeper/{data/myid,conf/zoo.cfg,conf/zookeeper-env.sh}

2.5 修改文件权限 ``所有节点``::

    $ chown -R root:root /opt/zookeeper
    $ chown -R zookep:zookep /data/zookeeper

三、修改配置
----------

3.1 生成myid文件 ``所有节点``::

    $ echo 1 > /data/zookeeper/data/myid    # VM01上操作
    $ echo 2 > /data/zookeeper/data/myid    # VM02上操作
    $ echo 3 > /data/zookeeper/data/myid    # VM03上操作

3.2 编辑配置文件 ``所有节点``:

.. code-block:: bash

    $ vim /data/zookeeper/conf/zoo.cfg
    # 添加如下内容:
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/data/zookeeper/data 
    dataLogDir=/data/zookeeper/data

    autopurge.purgeInterval=24
    autopurge.snapRetainCount=500

    clientPort=2181
    server.1=VM01:2888:3888
    server.2=VM02:2888:3888
    server.3=VM03:2888:3888

3.2 修改默认配置目录:
    
.. code-block:: bash

    $ vim /opt/zookeeper/bin/zkEnv.sh
    # 第25行加入如下内容
    ZOOCFGDIR=/data/zookeeper/conf

3.3 修改日志、PID目录:

.. code-block:: bash

    $ vim /data/zookeeper/conf/zookeeper-env.sh
    # 替换如下内容
    export JAVA_HOME=${JAVA_HOME}
    export ZOO_LOG_DIR=/data/zookeeper/logs
    export ZOOPIDFILE=/data/zookeeper/vars/run

四、启动程序
----------

4.1 启动应用程序 ``所有节点``::
    
二进制启动::

    $ cd /opt/zookeeper
    $ ZOOCFGDIR=/data/zookeeper/conf \
      ZOO_LOG_DIR=/data/zookeeper/logs \
      bin/zkServer.sh start

SysV启动脚本::

    $ 

supervisor启动配置:


.. note::
    
    选择一种启动方式即可，一般使用SysV启动脚本启动即可。

4.2 检测启动状态 ``所有节点``::

方法一:

.. code-block:: bash
    
    # Leader节点显示的状态
    $ /usr/local/zookeeper-3.4.6/bin/zkServer.sh status
    JMX enabled by default
    Using config: /usr/local/zookeeper-3.4.6/bin/../conf/zoo.cfg
    Mode: leader
    
    # Follower节点显示的状态
    $ /opt/zookeeper/bin/zkServer.sh status
    JMX enabled by default
    Using config: /opt/zookeeper/bin/../conf/zoo.cfg
    Mode: follower

方法二:

.. code-block:: bash

    $ echo stat | nc VM01 2181
    Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
    Clients:
     /192.168.182.101:38440[0](queued=0,recved=1,sent=0)

    Latency min/avg/max: 0/0/0
    Received: 37
    Sent: 36
    Connections: 1
    Outstanding: 0
    Zxid: 0x0
    Mode: follower
    Node count: 4

    $ echo stat | nc VM02 2181
    Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
    Clients:
     /192.168.182.101:34330[0](queued=0,recved=1,sent=0)

    Latency min/avg/max: 0/0/0
    Received: 9
    Sent: 8
    Connections: 1
    Outstanding: 0
    Zxid: 0x100000000
    Mode: follower
    Node count: 4

    $ echo stat | nc VM03 2181
    Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
    Clients:
     /192.168.182.101:47964[0](queued=0,recved=1,sent=0)

    Latency min/avg/max: 0/0/0
    Received: 4
    Sent: 3
    Connections: 1
    Outstanding: 0
    Zxid: 0x100000000
    Mode: leader
    Node count: 4



6 规范环境
----------

6.2 开机启动::

    ---
    
6.1 添加PATH:

.. code-block:: bash

    $ vim /etc/profile.d/zookeeper.sh
    # 添加如下内容:
    PATH=$PATH:/opt/zookeeper/bin
    export PATH
    $ source /etc/profile.d/zookeeper.sh


7 补充说明
----------

7.1 主要配置说明:

``dataDir``::

    这个目录为 Zookeeper 保存数据的目录用于保存myid和内存快照，默认情况下 Zookeeper 将写数据的事务日志文件也保存在这个目录里。

``dataLogDir``::

    事务日志目录，类似mysqlbinlog日志、redis的aof日志。

``autopurge.purgeInterval``::

    这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能。

``autopurge.snapRetainCount``::

    这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。

``tickTime``::

	这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
    
``clientPort``::

	这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
    
``initLimit``::

	这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒

``syncLimit``::
 
 	这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 5*2000=10 秒
    
``server.A=B:C:D``::

	其中 A 是一个数字（myid的内容），表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。
