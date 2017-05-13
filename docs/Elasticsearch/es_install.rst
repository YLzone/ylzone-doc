====================
[部署]-单机部署2.x.x
====================

1 背景介绍
----------------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    所需文件: elasticsearch-2.2.2.tar.gz
    操作用户: root
    运行用户: redis
    软件版本: elasticsearch-2.2.2
    部署位置: /opt/elasticsearch-2.2.2
    配置目录: /data/elasticsearch/conf
    数据目录: /data/elasticsearch/data
    脚本目录: /data/elasticsearch/data/scripts
    插件目录: /data/elasticsearch/data/plugins
    日志目录: /data/elasticsearch/logs
    PID 目录: /data/elasticsearch/run

1.2 功能说明::

    监听端口: 9200        # 对外服务的http端口    
    监听端口: 9300        # 节点间交互的tcp端口
    

2 解决依赖
----------

2.1 安装依赖组件::

    $ rpm -ivh jdk-8u60-linux-x64.rpm

2.2 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 9200 elastic  # 建立运行用户

2.3 创建所需目录::

    $ mkdir -pv /data/elasticsearch && cd /data/elasticsearch
    $ mkdir conf data logs run
    $ mkdir data/scripts /data/plugins
    $ chown -R root:root conf
    $ chown -R elastic:elastic data logs run

3 安装程序
----------

3.1 解压软件包::

    $ cd /tmp
    $ tar xf elasticsearch-2.2.2.tar.gz -C /opt

3.2 创建所需文件::   

    $ cp /opt/elasticsearch-2.2.2/config/* /data/elasticsearch/conf
    $ mv /opt/elasticsearch-2.2.2/config /opt/elasticsearch-2.2.2/config.default
    $ ln -sv /opt/elasticsearch-2.2.2/ /opt/elasticsearch
    $ ln -sv /data/elasticsearch/conf/ /opt/elasticsearch-2.2.2/config 

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash
    
    $ vim /data/elasticsearch/conf/elasticsearch.yml
    # 添加如下内容:
    # ======================== Elasticsearch Configuration =========================
    #
    # ---------------------------------- Cluster -----------------------------------
    #
    # Use a descriptive name for your cluster:
    #
    # cluster.name: my-application
    #
    # ------------------------------------ Node ------------------------------------
    #
    # Use a descriptive name for the node:
    #
    # node.name: node-1
    #
    # Add custom attributes to the node:
    #
    # node.rack: r1
    #
    # ----------------------------------- Paths ------------------------------------
    # Path to directory where to store the data (separate multiple locations by comma):
    path.data: /data/elasticsearch/data
    path.logs: /data/elasticsearch/logs
    path.scripts: /data/elasticsearch/data/scripts
    path.plugins: /data/elasticsearch/data/plugins

    # ----------------------------------- Memory -----------------------------------
    # bootstrap.mlockall: true

    # ---------------------------------- Network -----------------------------------
    network.host: 0.0.0.0
    http.port: 9200
    transport.tcp.port: 9300

    # --------------------------------- Discovery ----------------------------------
    # The default list of hosts is ["127.0.0.1", "[::1]"]
    # discovery.zen.ping.unicast.hosts: ["host1", "host2"]
    # discovery.zen.minimum_master_nodes: 3

    # ---------------------------------- Gateway -----------------------------------
    # gateway.recover_after_nodes: 3

    # ---------------------------------- Various -----------------------------------
    # node.max_local_storage_nodes: 1
    # action.destructive_requires_name: true
    index.number_of_shards: 1
    index.number_of_replicas: 0

4.2 主要配置说明:

``index.number_of_shards``:
    设置默认索引分片个数，默认为5片。这里为单机部署设置为1即可
    
``ndex.number_of_replicas``:
    设置默认索引副本个数，默认为1个副本。这里为单机部署设置为0即可

5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/elasticsearch/bin
    $ setsid su elastic -c "./elasticsearch -d -p /data/elasticsearch/run/elasticsearch.pid"

.. note::
    
    启动脚本会检测运行用户，禁止使用root用户启动。

5.2 SysV启动脚本::

    暂无

5.3 部署验证:

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


6 规范环境
----------

6.2 开机启动:

.. code-block:: bash
    
    $ vim /etc/rc.local
    # 添加如下内容
    $ setsid su elastic -c "./elasticsearch -d -p /data/elasticsearch/run/elasticsearch.pid"

6.3 添加PATH:

.. code-block:: bash

    $ vim /etc/profile.d/elasticsearch.sh
    # 添加如下内容:
    PATH=$PATH:/opt/elasticsearch/bin
    export PATH
    $ source /etc/profile.d/elasticsearch.sh
