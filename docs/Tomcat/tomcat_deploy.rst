=================
[部署]-部署tomcat
=================


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署类型: tar
    操作用户: root
    运行用户: tomcat
    软件版本: zookeeper-3.4.9
    部署位置: /opt/zookeeper
    配置目录: /data/zookeeper/conf
    数据目录: /data/zookeeper/data
    日志目录: /data/zookeeperlogs
    PID 目录: /data/zookeeper/run
    所需文件: apache-tomcat-7.0.72.tar
    
1.2 安全级别::

    权限级别: 高
    数据安全: 中

1.3 分布说明::

    192.168.182.101    VM01    zookeeper

1.4 端口介绍::

    2181: 表示和客户端通信的接口

1.5 操作命令::

    zookeeper-server
    zookeeper-client

..
   1.2 相关地址::
    下载地址
    ---
    智能安装: 
   1.3 关键命令::
    mysql mysqldump


2 解决依赖
----------

2.1 安装依赖组件::

    $ yum install jdk-8u60-linux-x64.rpm

2.2 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 2181 zookep

2.3 创建所需目录::

    $ mkdir -p /data/zookeeper && cd /data/zookeeper
    $ mkdir conf data logs run


2.4 修改hosts文件:

.. code-block:: bash

    $ vim /etc/hosts
    # 添加如下内容
    192.168.182.101 VM01
    
.. note::

    如果你使用了内部DNS服务器则不需要修改此文件，否则请按照操作实行。

3 安装程序
----------

3.1 解压软件包::

    $ cd /tmp
    $ tar xf zookeeper-3.4.9.tar.gz -C /opt
    $ ln -sv /opt/zookeeper-3.4.9/ /opt/zookeeper

3.2 创建所需文件::

    $ cp /opt/zookeeper/conf/* /data/zookeeper/conf
    $ mv /opt/zookeeper/conf /opt/zookeeper/conf.default
    $ ln -sv /data/zookeeper/conf/ /opt/zookeeper/conf
    $ touch /data/zookeeper/conf/zoo.cfg
    $ touch /data/zookeeper/data/myid

3.3 删除多余文件::
    
    $ rm -rf /opt/zookeeper/{recipes,src,docs}
    $ rm -f /opt/zookeeper/zookeeper-3.4.9.jar.{asc,md5,sha1}
    $ rm -f /opt/zookeeper/bin/{README.txt,*.cmd}

3.4 修改文件权限::

    $ chown -R zookep:zookep /data/zookeeper
    $ chown -R root:root /data/zookeeper/conf
    $ chown root:root /data/zookeeper


4 修改配置
----------

4.1 编辑配置文件:

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

5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/zookeeper && bin/zkServer.sh start

5.2 规范启动::

    $ cd /data/kafka && bin/zkServer.sh start

5.3 验证部署:

方法一:

.. code-block:: bash
    
    $ /usr/local/zookeeper-3.4.6/bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /opt/zookeeper/bin/../conf/zoo.cfg
    Mode: standalone

方法二:

.. code-block:: bash

    $ echo stat | nc VM01 2181
    Zookeeper version: 3.4.9-1757313, built on 08/23/2016 06:50 GMT
    Clients:
     /192.168.182.101:38558[0](queued=0,recved=1,sent=0)

    Latency min/avg/max: 0/0/0
    Received: 2
    Sent: 1
    Connections: 1
    Outstanding: 0
    Zxid: 0x0
    Mode: standalone
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
