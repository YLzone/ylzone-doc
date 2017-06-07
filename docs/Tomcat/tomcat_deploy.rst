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
    软件版本: tomcat-7.0.72
    所需文件: apache-tomcat-7.0.72.tar

1.2 节点说明::

    192.168.182.101    VM01    tomcat

1.3 目录说明::

    部署位置: /opt/tomcat7
    配置目录: /data/tomcat/.catalina-base/conf
    数据目录: /data/tomcat/.catalina-base/data
    日志目录: /data/tomcat/.catalina-base/logs
    PID 目录: /data/tomcat/.catalina-base/run
    
1.4 安全级别::

    权限级别: 高

1.5 端口说明::

    8080: 提供http服务的端口。

1.6 操作命令::

    tomcat: 服务启动及操作脚本。

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

    $ useradd -M -s /sbin/nologin -u 8080 tomcat

.. note::

    便于管理可以根据项目建立相关用户
    
    
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
    $ tar xf apache-tomcat-7.0.72.tar-C /opt
    $ ln -sv /opt/tomcat-7.0.72/ /opt/tomcat7

3.2 整理文件::

    $ rm -f /opt/tomcat7/{LICENSE,NOTICE,RELEASE-NOTES,RUNNING.txt}
    $ rm -rf /opt/tomcat7/{logs,temp,webapps,work}
    $ mv /opt/tomcat7/{conf,conf.default}

3.3 创建所需目录::

    $ mkdir /data/tomcat/.catalina-base{conf,webapps,logs,work,temp,data,local}
    $ ln -sv /data/tomcat/.catalina-base/webapps/ /data/tomcat/webapps
    $ ln -sv /data/tomcat/.catalina-base/logs/ /data/tomcat/logs
    $ ln -sv /data/tomcat/.catalina-base/data/ /data/tomcat/data
    $ ln -sv /data/tomcat/.catalina-base/local/ /data/tomcat/local

3.4 创建所需文件::
    
   $ cp /opt/tomcat7/conf/* /data/tomcat/.catalina-base/conf
   $ touch /data/tomcat/.catalina-base/tomcat.sh
   $ echo "Index Successful!" > /data/tomcat/.catalina-base/webapps/ROOT/index.html

3.5 修改文件权限::

    $ chown -R tomcat:tomcat /opt/tomcat7
    $ chown -R tomcat:tomcat /data/tomcat/.catalina-base
    $ chown -R root:root /data/tomcat/.catalina-base/conf

.. note::

    为了便于管理和安全考虑，权限用户可分为 ``部署用户`` ``配置用户`` ``运行用户`` 三类。

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    $ vim /data/zookeeper/conf/zoo.cfg
    # 添加如下内容:
    <?xml version='1.0' encoding='utf-8'?>

    <Server port="-1" shutdown="SHUTDOWN">
      <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
      <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
      <Listener className="org.apache.catalina.core.JasperListener" />
      <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
      <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
      <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
    
      <GlobalNamingResources>
        <Resource name="UserDatabase" auth="Container"
                  type="org.apache.catalina.UserDatabase"
                  description="User database that can be updated and saved"
                  factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
                  pathname="conf/tomcat-users.xml" />
      </GlobalNamingResources>
    
      <Service name="Catalina">
    
                 <!-- acceptCount="2000" -->
        <Connector port="8080" protocol="HTTP/1.1"
                   acceptCount="1024"
                   minSpareThreads="50"
                   maxThreads="1020"
                   connectionTimeout="20000"
                   redirectPort="8443"
                   enableLookups="false"
                   useBodyEncodingForURI="true"
                   URIEncoding="UTF-8" />
    
        <Engine name="Catalina" defaultHost="localhost">
          <Realm className="org.apache.catalina.realm.LockOutRealm">
            <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                   resourceName="UserDatabase"/>
          </Realm>
    
          <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
            <Context path="" docBase="webapps/ROOT"/>
            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                   prefix="localhost_access_log." suffix=".txt"
                   pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    
          </Host>
        </Engine>
      </Service>
    </Server>

4.2 修改日志配置::

    $ sed -i '/^handlers =/ s/^/#/' /data/tomcat/.catalina-base/conf/logging.properties

5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/tomcat
    $ CATALINA_BASE=/data/tomcat/.catalina-base \ 
      CATALINA_PID=/data/tomcat/.catalina-base/run \
      bin/catalina.sh start

5.2 规范启动::

    $ cd /data/kafka && bin/zkServer.sh start

5.3 验证部署:

方法一:

.. code-block:: bash
    
    $ curl http://127.0.0.1:8080
    Index Successful!

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
