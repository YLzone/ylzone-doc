========================
[部署]-集群部署zookeeper
========================


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    操作用户: root
    运行用户: zookep
    软件版本: zookeeper-3.4.9
    部署位置: /opt/zookeeper
    配置目录: /data/zookeeper/conf
    数据目录: /data/zookeeper/data
    日志目录: /data/zookeeperlogs
    PID 目录: /data/zookeeper/run
    所需文件: zookeeper-3.4.9.tar.gz
    
1.2 安全级别::

    权限级别: 高
    数据同步: 中

1.3 分布说明::

    192.168.182.101    VM01    zookeeper
    192.168.182.102    VM02    zookeeper
    192.168.182.103    VM03    zookeeper

1.4 端口介绍::

    2181: 表示和客户端通信的接口
    2888: 表示集群间交换信息的端口；
    3888: 表示集群选取"Ladaer"的通信端口。

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

2.3 创建所需文件::

    $ mkdir -p /data/zookeeper && cd /data/zookeeper
    $ mkdir conf data logs run
    $ chown -R zookep:zookep /data/zookeeper
    $ chown root:root /data/zookeeper
    $ chown -R root:root /data/zookeeper/conf
    $ ls -sv /data/zookeeper/conf/ /opt/zookeeper/conf

2.4 修改hosts文件 ``所有节点`` :

.. code-block:: bash

    $ vim /etc/hosts
    # 添加如下内容
    192.168.182.101 VM01
    192.168.182.102 VM02
    192.168.182.103 VM03
    
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

3.3 删除多余文件::
    
    $ rm -rf /opt/zookeeper/recipes
    $ rm -rf /opt/zookeeper/src
    $ rm -rf /opt/zookeeper/docs
    $ rm -f /opt/zookeeper/zookeeper-3.4.9.jar.{asc,md5,sha1}
    $ rm -f /opt/zookeeper/bin/README.txt
    $ rm -f /opt/zookeeper/bin/*.cmd


4 修改配置
----------

4.1 生成myid文件::

    echo 1 > /data/zookeeper/data/myid    # VM01上操作
    echo 2 > /data/zookeeper/data/myid    # VM02上操作
    echo 3 > /data/zookeeper/data/myid    # VM03上操作

4.2 编辑配置文件 ``VM01上操作`` :

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

4.3 同步配置文件 ``VM01上操作`` ::

    scp /data/zookeeper/conf/zoo.cfg VM02:/data/zookeeper/conf/zoo.cfg
    scp /data/zookeeper/conf/zoo.cfg VM03:/data/zookeeper/conf/zoo.cfg


5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/zookeeper
    $ zkServer.sh start

5.2 规范启动::

    $ cd /data/kafka
    $ ./kafka start

5.3 验证部署:

.. code-block:: bash

    # 创建一个topic
    $ cd /opt/kafka
    $ bin/kafka-topics.sh --create --zookeeper VM01:2181,VM02:2181,VM03:2181/kafka --replication-factor 1 --partitions 1 --topic  test
    
    # 查看创建的topic
    $ bin/kafka-topics.sh --list --zookeeper VM01:2181,VM02:2181,VM03:2181/kafka

    # 启动一个消费者
    $ bin/kafka-console-consumer.sh --zookeeper  ZKF1.S0001.WJ-KF-B.BJ.JRX:2181/kafka --topic test 

    # 启动一个生产者(在另一个终端中)
    $ bin/kafka-console-producer.sh --broker-list note:9092 --topic test
    hello world       <== 输入信息
    
    # 当在生产者终端中输入信息后，此信息应该会出现在消费者终端，否则为异常。


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

``dataDir`` ::

    这个目录为 Zookeeper 保存数据的目录用于保存myid和内存快照，默认情况下 Zookeeper 将写数据的事务日志文件也保存在这个目录里。

``dataLogDir`` ::

    事务日志目录，类似mysqlbinlog日志、redis的aof日志。

``autopurge.purgeInterval`` ::

    这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能。

``autopurge.snapRetainCount`` ::

    这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。

``tickTime`` ::

	这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
    
``clientPort`` ::

	这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
    
``initLimit`` ::

	这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒

``syncLimit`` ::
 
 	这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 5*2000=10 秒
    
```server.A=B:C:D`` ::

	其中 A 是一个数字（myid的内容），表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。