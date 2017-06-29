====================
[安装]-编译python2.7
====================


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署类型: tar
    操作用户: root
    运行用户: tomcat
    软件版本: tomcat-7.0.72
    所需文件: Python-2.7.9.tgz
              setuptools-21.2.1.tar.gz
              pip-8.1.2.tar.gz

1.2 目录说明::

    部署位置: /usr/local/python2.7.9
    
1.3 下载地址::
    
    Python     下载地址: https://www.python.org/ftp/python/
    Setuptools 下载地址: https://pypi.python.org/pypi/setuptools#downloads
    Pip        下载地址: https://pypi.python.org/pypi/pip/

2 解决依赖
----------

2.1 安装依赖组件::

    $ yum install gcc zlib-devel openssl-devel readline-devel sqlite-devel

``zlib-devel``:

    解决 setuptools："Compression requires the (missing) zlib module"

``openssl-devel``:

    解决 from urllib2 import (Request, urlopen, URLError, HTTPErrorImportError: cannot import name HTTPSHandler

``readline-devel``:

    解决 交互无法直接删除问题

``sqlite-devel``:

    解决 Sqlite依赖库问题


3 编译程序
----------

3.1 获取软件包::

    $ cd /tmp
    $ wget https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tgz

3.2 编译程序::

    $ tar xf Python-2.7.9.tgz
    $ cd Python-2.7.9
    $ ./configure --prefix=/usr/local/python2.7.9
    $ make && make install



3.3 创建所需目录::

    $ mkdir -pv /data/tomcat/.catalina-base/{conf,webapps,logs,work,temp,data,local}
    $ mkdir -v /data/tomcat/.catalina-base/webapps/ROOT
    $ ln -sv /data/tomcat/.catalina-base/webapps/ /data/tomcat/webapps
    $ ln -sv /data/tomcat/.catalina-base/logs/ /data/tomcat/logs
    $ ln -sv /data/tomcat/.catalina-base/data/ /data/tomcat/data
    $ ln -sv /data/tomcat/.catalina-base/local/ /data/tomcat/local

3.4 创建所需文件::
    
   $ cp /opt/tomcat7/conf.default/* /data/tomcat/.catalina-base/conf
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

    $ vim /data/tomcat/.catalina-base/conf/server.xml
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
            <Context path="" docBase="ROOT"/>
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
    
    $ cd /opt/tomcat7
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
