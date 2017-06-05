==================
[部署]-部署dnsmasq
==================


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: 
    软件版本: 2.48-17.el6
    部署位置: 
    配置目录: /etc/dnsmasq.conf /etc/dnsmasq.d
    数据目录: 不生成数据
    日志目录: 
    PID 目录: /data/zookeeper/run
    所需文件: dnsmasq-2.48-17.el6.x86_64

1.4 端口介绍::

    2181: 表示和客户端通信的接口
    2888: 表示集群间交换信息的端口；
    3888: 表示集群选取"Ladaer"的通信端口。

1.5 操作命令::


..
   1.2 相关地址::
    下载地址
    ---
    智能安装: 
    mysql mysqldump


2 解决依赖
----------

2.1 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 53 dnsmasq

2.1 创建所需目录::

    $ mkdir -p /data/dnsmasq && cd /data/dnsmasq
    $ mkdir conf logs

3 安装程序
----------

3.1 使用yum安装程序::

    $ yum install dnsmasq -y

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    $ ln -sv /data/dnsmasq/conf/dnsmasq.conf /etc/dnsmasq.conf
    $ vim /etc/dnsmasq.conf
    # 添加如下内容:
    conf-file=/data/dnsmasq/conf/dnsmasq.conf

    $ vim /data/dnsmasq/conf/dnsmasq.conf
    # 添加如下内容
    user=dnsmasq
    group=dnsmasq
    log-facility=/data/dnsmasq/logs/dnsmasq.log

    strict-order
    resolv-file=/data/dnsmasq/data/dnsmasq.resolv.conf

    no-hosts
    addn-hosts=/data/dnsmasq/data//dnsmasq.hosts.conf

    $ echo "nameserver 114.114.114.114" >> /data/dnsmasq/data/dnsmasq.resolv.conf

.. note::
    
    使用data下的配置文件便于服务迁移，如果要新加入解析可以修改 ``dnsmasq.hosts.conf`` 内容。

5 运行程序
----------

5.1 启动命令 ``所有节点``::
    
    $ service dnsmasq start


5.3 验证部署 ``所有节点``: 

方法一:

.. code-block:: bash
    
   修改客户端为此服务器
   dig 

方法二:

.. code-block:: bash


6 规范环境
----------

6.1 开机启动::

    chkconfig dnsmasq on
    
6.2 日志轮转::

    log

7 补充说明
----------

7.1 主要配置说明:
