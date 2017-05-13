====================
[安装]-单机部署2.x.x
====================

1 安装组件
----------------

1.0 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    所需文件: elasticsearch-2.2.2.tar.gz
    操作用户: root
    运行用户: zookeeper
    软件版本: zookeeper-3.4.6
    部署位置: /opt/zookeeper-3.4.6
    配置目录: /data/zookeeper/conf
    数据目录: /data/zookeeper/data
    日志目录: /data/zookeeper/logs
    脚本目录: /data/elasticsearch/scripts
    插件目录: /data/elasticsearch/plugins
    特殊说明: 9200端口为对外服务的http端口 9300端口为节点间交互的tcp端口

1.1 安装JDK依赖::

    $ rpm -ivh jdk-8u60-linux-x64.rpm

1.2 解压安装::

    $ cd /tmp
    $ tar xf zookeeper-3.4.6.tar.gz -C /opt
    $ ln -sv /opt/zookeeper-3.4.6/ /opt/zookeeper

1.3 规范运行用户::

    $ useradd -M -s /sbin/nologin -u 2181 zookeeper  # 建立运行用户

1.4 规范目录创建::
    
    # 创建必要目录
    $ mkdir -pv /data/zookeeper
    $ cd /data/zookeeper
    $ mkdir conf data logs run

    # 创建必要文件
    $ cd /opt/zookeeper
    $ cp ./conf/* /data/zookeeper/conf
    $ mv ./conf ./config.default
 
1.5 规范目录权限::

    $ chown -R root:root /opt/elasticsearch-2.2.2
    $ chown -R root:root /data//elasticsearch/conf
    $ chown -R elastic:elastic /data/elasticsearch/data 
    $ chown -R elastic:elastic /data/elasticsearch/logs
    $ chown -R elastic:elastic /data/elasticsearch/run
    $ chown -R elastic:elastic /data/elasticsearch/plugins
    $ chown -R elastic:elastic /data/elasticsearch/scripts
    $ ln -sv /data/elasticsearch/conf/ /opt/elasticsearch-2.2.2/config 

1.6 修改配置文件:

.. code-block:: bash
    
    # vim /data/zookeeper/conf/zoo.cfg
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/data/zookeeper/data
    clientPort=2181

主要配置说明:

``index.number_of_shards``:
    设置默认索引分片个数，默认为5片。这里为单机部署设置为1即可
    
``ndex.number_of_replicas``:
    设置默认索引副本个数，默认为1个副本。这里为单机部署设置为0即可

1.7 启动命令::
    
    # 守护进程方式启动
    $ cd /opt/elasticsearch/bin
    $ su elastic -c "./elasticsearch -d -p /data/elasticsearch/run/elasticsearch.pid"

    # 查看帮助信息
    $ ./elasticsearch --hlep

验证测试:

.. code-block:: bash

    $ curl 127.0.0.1:9200
    {
      "name" : "Colossus",
      "cluster_name" : "elasticsearch",
      "version" : {
        "number" : "2.2.2",
        "build_hash" : "fcc01dd81f4de6b2852888450ce5a56436fd5852",
        "build_timestamp" : "2016-03-29T08:49:35Z",
        "build_snapshot" : false,
        "lucene_version" : "5.4.1"
      },
      "tagline" : "You Know, for Search"
    }

1.8 创建启动脚本/etc/init.d/redis_6379:


1.9 使用脚本启动::

    $ chmod +x /etc/init.d/redis_6379
    $ service redis_6379 start
    $ chkconfig --add redis_6379
    $ chkconfig redis_6379 on

1.10 添加环境变量::

    编辑配置文件:
    $ vim /etc/profile.d/redis.sh

    添加如下内容:
    PATH=$PATH:/opt/redis/bin 
    export PATH

    载入环境变量:
    $ source /etc/profile.d/redis.sh
