===================
[部署]-集群部署HDFS
===================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: tar
    操作用户: root
    运行用户: hdfs
    软件版本: hadoop
    所需文件: hadoop-2.7.3.tar.gz, jdk-8u60-linux-x64.gz

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - NameNode, SecondaryNameNode, DataNode
  * - VM02
    - 192.168.182.102
    - DataNode
  * - VM03
    - 192.168.182.103
    - DataNode
    
0.3 目录说明::

    程序目录: /opt/hadoop
    配置目录: /data/hadoop/conf
    数据目录: /data/hadoop/data/hdfs/{name,data,namesecondary}
    日志目录: /data/hadoop/logs
    临时目录: /data/hadoop/vars/tmp
    PID 目录: /data/hadoop/vars/run

0.4 端口说明::

    8020:  namenode 监听的端口，HDFS的入口。
    50070: namenode 管理页面（Web UI）。 

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进入，蓝色为说明提示。



一、解决依赖
------------

..
    加入环境检测
    1. 检测jdk版本，删除不兼容jdk
    2. 检测主机名对应关系
    3. 时间检测，检查时间是否同步，配置NTP
    4. 存储空间检测，检查空间是否满足要求
    5. 文件、进程打开数

1.1 安装依赖组件:

安装jdk::

    # 安装软件包
    $ mkdir -v /usr/java
    $ tar xvf jdk-8u60-linux-x64.gz -C /usr/java
    $ ln -sv /usr/java/jdk1.8.0_60 /usr/java/latest
    $ ln -sv /usr/java/latest /usr/java/default
    $ chown -R root:root /usr/java/jdk1.8.0_60
    $ echo 'export JAVA_HOME=/usr/java/default' > /etc/profile.d/java.sh
    $ echo 'export PATH=${PATH}:${JAVA_HOME}/bin' >> /etc/profile.d/java.sh
    $ source /etc/profile.d/java.sh

    # 验证安装，显示如下内容表示成功。
    $ java -version
    java version "1.8.0_60"
    Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
    Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)

1.2 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 8020 hdfs


二、安装程序
------------

2.1 解压缩软件包::

    $ cd /tmp
    $ tar xf hadoop-2.7.3.tar.gz -C /opt
    $ mv -v /opt/hadoop-2.7.3/ /opt/hadoop
    $ echo "version: hadoop-2.7.3" >> /opt/hadoop/VERSION.md

2.2 整理程序目录::

    $ mv -v /opt/hadoop/etc /opt/hadoop/etc.orig
    $ rm -fv /opt/hadoop/*.txt
    $ rm -fv /opt/hadoop/bin/*.cmd
    $ rm -fv /opt/hadoop/sbin/*.cmd
    $ rm -fv /opt/hadoop/libexec/*.cmd
    $ rm -fv /opt/hadoop/etc.orig/hadoop/*.cmd
    $ rm -rf /opt/hadoop/share/doc

2.3 创建所需目录::

    $ mkdir -pv /data/hadoop/{conf,data,logs,vars}
    $ mkdir -pv /data/hadoop/data/hdfs/{name,namesecondary,data}
    $ mkdir -pv /data/hadoop/vars/run

2.4 创建所需文件::

    $ cp /opt/hadoop/etc.orig/hadoop/* /data/hadoop/conf

2.5 修改文件权限::

    $ chown -R root:root /opt/hadoop
    $ chown -R hdfs:hdfs /data/hadoop/data/hdfs
    $ chown -R hdfs:hdfs /data/hadoop/{logs,vars}
    

2.6 修改环境变量::

    $ echo "PATH=$PATH:/opt/hadoop/bin" > /etc/profile.d/hadoop.sh
    $ echo "export PATH" >> /etc/profile.d/hadoop.sh
    $ source /etc/profile.d/hadoop.sh                # 执行此命令使如上配置生效

2.7 设置开机启动::
    
    # NameNode 开机启动
    $ sed -i '6i su hdfs -s /bin/bash -c "/opt/hadoop/sbin/hadoop-daemon.sh --config /data/hadoop/conf start namenode"' /etc/rc.d/rc.local

    # DataNode 开机启动
    $ sed -i '7i su hdfs -s /bin/bash -c "/opt/hadoop/sbin/hadoop-daemon.sh --config /data/hadoop/conf start datanode"' /etc/rc.d/rc.local

    # SecondaryNamenode 开机启动
    $ sed -i '8i su hdfs -s /bin/bash -c "/opt/hadoop/sbin/hadoop-daemon.sh --config /data/hadoop/conf start secondarynamenode"' /etc/rc.d/rc.local

.. warning::

    上述三个开机启动，请根据规划参照标注提示的指定节点操作。如果后续准备使用 supervisor 启动，则不要执行 ``2.7步骤``。


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: xml

    $ vim /data/hadoop/conf/core-site.xml
    # 替换如下内容:
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

    <configuration>

    <!-- HDFS工作目录的设置,默认是linux的/temp,每次linux重启会清空,hadoop中的数据会全部丢失. -->
    <!-- 其它一些目录是以这个临时目录为基本目录的,如dfs.name.dir和dfs.name.edits.dir等. -->
    <!-- 用来指定使用hadoop时产生文件的存放目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:///data/hadoop</value>
    </property>

    <!-- 配置 namenode 的端口及所在位置,也可说是HDFS的入口 -->
    <!-- ***** 注意此配置为 namenode 节点配置，就按规划修改此地址 ***** -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://VM01:8020</value>
    </property>

    <!-- The number of seconds between two periodic checkpoints. -->
    <property>  
        <name>fs.checkpoint.period</name>  
        <value>3600</value>  
    </property>  
     
    <!-- The size of the current edit log (in bytes) that triggers  
         a periodic checkpoint even if the fs.checkpoint.period hasn't expired. -->
    <property>  
        <name>fs.checkpoint.size</name>  
        <value>67108864</value>  
    </property> 

    </configuration>

.. code-block:: xml

    $ vim /data/hadoop/conf/hdfs-site.xml
    # 替换如下内容:
    <?xml version="1.0" encoding="UTF-8"?>
    <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

    <configuration>

    <!-- namenode 所使用的元数据保存，一般建议在nfs上保留一份，也可以在一台服务器的多块硬盘上使用 -->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///data/hadoop/data/hdfs/name</value>
    </property>

    <!-- The address and the base port where the dfs namenode web ui will listen on.
         If the port is 0 then the server will start on a free port. -->
    <!-- ***** 注意此配置为 namenode 节点配置，就按规划修改此地址 ***** -->
    <property>
        <name>dfs.http.address</name>
        <value>VM01:50070</value>
    </property>

    <!-- secondary namenode 节点存储 checkpoint 文件目录 -->
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>file:///data/hadoop/data/hdfs/namesecondary</value>
    </property>

    <!-- ***** 注意此配置为 secondarynamenode 节点配置，就按规划修改此地址 ***** -->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>VM02:50090</value>
    </property>

    <!-- 真正的datanode数据保存路径，可以写多块硬盘，逗号分隔。
         把这些位置分散在每个节点上的所有磁盘上可以实现磁盘 I/O 平衡，因此会显著改进磁盘 I/O 性能。-->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///data/hadoop/data/hdfs/data</value>
    </property>

    <!-- 指定dfs保存数据的副本数量 -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

    </configuration>

3.2 修改默认配置目录::
    
    $ sed -i '20i HADOOP_CONF_DIR=/data/hadoop/conf' /opt/hadoop/libexec/hdfs-config.sh

3.3 修改日志、PID目录::

    $ echo "export HADOOP_LOG_DIR=/data/hadoop/logs" >> /data/hadoop/conf/hadoop-env.sh
    $ echo "export HADOOP_PID_DIR=/data/hadoop/vars/run" >> /data/hadoop/conf/hadoop-env.sh

3.4 修改JAVA_HOME环境变量::

    $ echo 'export JAVA_HOME=${JAVA_HOME:-"/usr/java/default"}' >> /data/hadoop/conf/hadoop-env.sh
    

四、启动程序
------------

4.1 启动之前操作:

初始化NameNode数据::

    $ su -s /bin/bash hdfs -c "hdfs --config /data/hadoop/conf namenode -format"

.. warning::

    此步骤只在 ``NameNode`` 操作，功能为初始化数据。仅在第一次启动之前操作即可，后续启动不要执行此操作。

4.2 启动应用程序:
    
二进制启动::

    # NameNode 启动
    $ cd /opt/hadoop/sbin
    $ su hdfs -s /bin/bash -c "./hadoop-daemon.sh --config /data/hadoop/conf start namenode"

    # DataNode 启动
    $ cd /opt/hadoop/sbin
    $ su hdfs -s /bin/bash -c "./hadoop-daemon.sh --config /data/hadoop/conf start datanode"

    # SecondaryNamenode 启动
    $ cd /opt/hadoop/sbin
    $ su hdfs -s /bin/bash -c "./hadoop-daemon.sh --config /data/hadoop/conf start secondarynamenode"
    
.. note::

    请根据规划参照标注提示指定节点操作。运行是可以用参数 ``--config`` 指定配置目录，如果不指定则使用 ``3.2步骤`` 所配置的目录。

SysV启动脚本::

    # NameNode 启动:
    $ service namenode start

    # DataNode 启动:
    $ service datanode start

    # SecondaryNamenode 启动:
    $ service secondarynamenode start

.. note::

    请根据规划参照标注提示指定节点操作。使用SysV脚本启动需要 ``redhat-lsb-core`` 此程序包，请提前安装。安装命令 ``yum install redhat-lsb-core``

supervisor启动配置:

.. code-block:: bash

    [program:mysql]
    command=/usr/local/python2.7.9/bin/pidproxy /data/mysql/data/mysqld.pid
     /opt/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
    stdout_logfile=/tmp/mysql.log
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10

.. warning::
    
    选择一种启动方式即可，一般使用SysV启动脚本启动即可。

4.2 检测启动状态::

    $ su hdfs -s /bin/bash -c "jps -l"

4.3 启动后续操作:

创建HDFS临时目录::
    
    $ hdfs dfs -mkdir /tmp
    $ hdfs dfs -chown -R 1777 /tmp


五、附属功能
------------

5.1 环境规范操作::

    # 暂无
