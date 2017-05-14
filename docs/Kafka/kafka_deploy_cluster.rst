====================
[部署]-集群部署kafka
====================

1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    操作用户: root
    运行用户: kafka
    软件版本: kafka_2.11-0.10.0.0
    部署位置: /opt/kafka
    配置目录: /data/kafka/conf
    数据目录: /data/kafka/data
    日志目录: /data/kafka/logs
    PID 目录: /data/kafka/run
    所需文件: kafka_2.11-0.10.0.0.tgz、zookeeper-3.4.9.tar.gz
    
1.2 分布说明::

    192.168.182.101    node1    zookeeper kafka
    192.168.182.102    node2    zookeeper kafka
    192.168.182.103    node3    zookeeper kafka


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

2.1 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 9092 kafka

2.3 创建所需目录::

    $ mkdir -p /data/kafka && cd /data/kafka
    $ mkdir conf data logs temp run
    $ chown -R kafka:kafka /data/mysql
    $ chown -R root:root /data/kafka/conf

3 安装程序
----------

3.1 解压软件包::

    $ cd /tmp
    $ tar xf kafka_2.11-0.10.0.0.tgz -C /opt
    $ ln -sv /opt/kafka_2.11-0.10.0.0 /opt/kafka

3.2 创建所需文件::

    $ cp /opt/kafka/conf/* /data/kafka/conf
    $ mv /opt/kafka/conf /opt/kafka/conf.default

3.3 删除多余文件::
    
    $ rm -rf /opt/kafka/bin/windows

.. node::

    /opt/kafka/bin/windows 此目录下为windows下所用脚本在linux中无用可删除

4 修改配置
----------

4.1 编辑配置文件 ``node1上操作`` :

.. code-block:: bash

    $ vim /data/kafka/conf/server.properties
    # 添加如下内容:
    ############################# Server Basics #############################
    broker.id=101

    ############################# Socket Server Settings #############################
    listeners=PLAINTEXT://:9092
    port=9092
    host.name=192.168.182.101

    num.network.threads=3
    num.io.threads=8

    socket.send.buffer.bytes=102400
    socket.receive.buffer.bytes=102400
    socket.request.max.bytes=104857600

    ############################# Log Basics #############################
    log.dirs=/data/kafka/data

    num.partitions=1
    num.recovery.threads.per.data.dir=1

    ############################# Log Flush Policy #############################
    #log.flush.interval.messages=10000
    #log.flush.interval.ms=1000

    ############################# Log Retention Policy #############################
    log.retention.hours=1
    log.segment.bytes=1073741824
    log.retention.check.interval.ms=300000

    ############################# Zookeeper #############################
    zookeeper.connect=node1:2181,node2:node3:2181/kafka
    zookeeper.connection.timeout.ms=6000
    delete.topic.enable=true

4.2 编辑配置文件 ``node2上操作`` :

.. code-block:: bash

    $ vim /data/kafka/conf/server.properties
    # 添加如下内容:
    ############################# Server Basics #############################
    broker.id=102

    ############################# Socket Server Settings #############################
    listeners=PLAINTEXT://:9092
    port=9092
    host.name=192.168.182.102

    num.network.threads=3
    num.io.threads=8

    socket.send.buffer.bytes=102400
    socket.receive.buffer.bytes=102400
    socket.request.max.bytes=104857600

    ############################# Log Basics #############################
    log.dirs=/data/kafka/data

    num.partitions=1
    num.recovery.threads.per.data.dir=1

    ############################# Log Flush Policy #############################
    #log.flush.interval.messages=10000
    #log.flush.interval.ms=1000

    ############################# Log Retention Policy #############################
    log.retention.hours=1
    log.segment.bytes=1073741824
    log.retention.check.interval.ms=300000

    ############################# Zookeeper #############################
    zookeeper.connect=node1:2181,node2:node3:2181/kafka
    zookeeper.connection.timeout.ms=6000
    delete.topic.enable=true
 
4.3 编辑配置文件 ``node3上操作`` :

.. code-block:: bash

    $ vim /data/kafka/conf/server.properties
    # 添加如下内容:
    ############################# Server Basics #############################
    broker.id=103

    ############################# Socket Server Settings #############################
    listeners=PLAINTEXT://:9092
    port=9092
    host.name=192.168.182.103

    num.network.threads=3
    num.io.threads=8

    socket.send.buffer.bytes=102400
    socket.receive.buffer.bytes=102400
    socket.request.max.bytes=104857600

    ############################# Log Basics #############################
    log.dirs=/data/kafka/data

    num.partitions=1
    num.recovery.threads.per.data.dir=1

    ############################# Log Flush Policy #############################
    #log.flush.interval.messages=10000
    #log.flush.interval.ms=1000

    ############################# Log Retention Policy #############################
    log.retention.hours=1
    log.segment.bytes=1073741824
    log.retention.check.interval.ms=300000

    ############################# Zookeeper #############################
    zookeeper.connect=node1:2181,node2:node3:2181/kafka
    zookeeper.connection.timeout.ms=6000
    delete.topic.enable=true

5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/mysql/bin
    $ su kafka -c "setsid bin/kafka-server-start.sh config/server.properties &> /data/kafka/logs/kafka.out"

5.2 规范启动::

    $ cd /data/kafka
    $ ./kafka start

5.3 验证部署:

.. code-block:: bash

    # 创建一个topic
    $ cd /opt/kafka
    $ bin/kafka-topics.sh --create --zookeeper node1:2181,node2:2181,node3:2181/kafka --replication-factor 1 --partitions 1 --topic  test
    
    # 查看创建的topic
    $ bin/kafka-topics.sh --list --zookeeper node1:2181,node2:2181,node3:2181/kafka

    # 启动一个消费者
    $ bin/kafka-console-consumer.sh --zookeeper  ZKF1.S0001.WJ-KF-B.BJ.JRX:2181/kafka --topic test 

    # 启动一个生产者(在另一个终端中)
    $ bin/kafka-console-producer.sh --broker-list note:9092 --topic test
    hello world       <== 输入信息
    
    # 当在生产者终端中输入信息后，此信息应该会出现在消费者终端，否则为异常。


6 规范环境
----------

6.2 开机启动::

    $ chkconfig --add mysql
    $ chkconfig mysql on

6.3 添加PATH:

.. code-block:: bash

    $ vim /etc/profile.d/mysql.sh
    # 添加如下内容:
    PATH=$PATH:/opt/mysql/bin
    export PATH
    $ source /etc/profile.d/mysql.sh

7 补充说明

7.1 主要配置说明:

``broker.id`` ::

    broker的id在集群中的唯一正整数标示

``listeners`` ::

    ...

``port`` ::

    server接受客户端连接的端口

``host.name`` ::

    ...

``log.dirs`` ::

    接收消息的数据目录

``zookeeper.connect``::

    zookeeper的连接地址,格式为 ``node1:port,node2:port/chroot ``
