=================
[部署]-部署tomcat
=================


零、背景介绍
----------

0.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署类型: tar
    操作用户: root
    运行用户: tomcat
    软件版本: tomcat-7.0.72
    所需文件: apache-tomcat-7.0.72.tar

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - tomcat

0.3 目录说明::

    部署位置: /opt/tomcat7
    配置目录: /data/tomcat7/conf
    数据目录: /data/tomcat7/data
    日志目录: /data/tomcat7/logs
    PID 目录: /data/tomcat7/vars/run

0.4 端口说明::

    8080: 提供http服务的端口。
    18080: 提供JMX服务的端口。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正确部署，蓝色为说明提示。


一、解决依赖
----------

1.1 安装依赖组件::

安装JDK::

    $ mkdir /usr/java
    $ tar xf jdk-8u60-linux-x64.gz -C /usr/java
    $ ln -sv /usr/java/jdk1.8.0_60 /usr/java/latest
    $ ln -sv /usr/java/latest /usr/java/default
    $ chown -R root:root /usr/java/jdk1.8.0_60
    $ echo 'export JAVA_HOME=/usr/java/default' > /etc/profile.d/java.sh
    $ echo 'export PATH=${PATH}:${JAVA_HOME}/bin' >> /etc/profile.d/java.sh
    $ source /etc/profile.d/java.sh

1.2 创建运行用户::

    $ useradd -s /sbin/nologin -u 8080 tomcat

.. note::

    便于管理可以根据项目建立相关用户

1.3 修改hosts文件:

.. code-block:: bash

    $ vim /etc/hosts
    # 添加如下内容
    192.168.182.101 VM01
    
.. note::

    如果你使用了内部DNS服务器则不需要修改此文件，否则请按照操作实行。

1.4 配置时间同步 ``所有节点``::

    $ crontab -e
    # 每两小时 Linux 系统就会自动的进行网络时间校准
    00 */2 * * * root /usr/sbin/ntpdate cn.pool.ntp.org

1.5 修改资源限制 ``所有节点``:

.. code-block:: bash

    $ vim /etc/security/limits.d/90-nofile.conf
    # 添加如下内容:
    tomcat          soft    nofile     65535
    tomcat          hard    nofile     65535

    $ vim /etc/security/limits.d/90-nproc.conf
    # 添加如下内容:
    tomcat          soft    nproc     unlimited
    tomcat          hard    nproc     unlimited


二、安装程序
----------

2.1 解压软件包::

    $ cd /tmp
    $ tar xf apache-tomcat-7.0.72.tar.gz -C /opt
    $ mv /opt/apache-tomcat-7.0.72/ /opt/tomcat7

2.2 整理文件::

    $ mv /opt/tomcat7/{conf,conf.orig}
    $ rm -fv /opt/tomcat7/{LICENSE,NOTICE,RELEASE-NOTES,RUNNING.txt}
    $ rm -fv /opt/tomcat7/bin/*.bat
    $ rm -rfv /opt/tomcat7/{logs,temp,webapps,work}

2.3 创建所需目录::

    $ mkdir -pv /data/tomcat7/{bin,conf,data,logs,vars}
    $ mkdir -pv /data/tomcat7/vars/{run,tmp,apps/ROOT,work}

2.4 创建所需文件::
    
   $ cp /opt/tomcat7/conf.orig/* /data/tomcat7/conf
   $ echo "Index Successful!" > /data/tomcat7/vars/apps/ROOT/index.html

2.5 修改文件权限::

    $ chown -R root:root /opt/tomcat7
    $ chown -R tomcat:tomcat /data/tomcat7

.. note::

    为了便于管理和安全考虑，权限用户可分为 ``部署用户`` ``配置用户`` ``运行用户`` 三类。

2.6 设置开机启动::

    $ sed -i '6i su tomcat -s /bin/bash -c "/opt/tomcat7/bin/startup start"' /etc/rc.d/rc.local

.. warning::

    如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。


三、修改配置
----------

3.1 编辑配置文件:

.. code-block:: xml

    $ vim /data/tomcat7conf/server.xml
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
    
          <Host name="localhost"  appBase="vars/apps" 
                unpackWARs="true" autoDeploy="true" workDir="vars/work">

            <!-- Context path="" docBase="vars/apps/ROOT"/ -->
            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                   prefix="localhost_access_log." suffix=".txt"
                   pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    
          </Host>
        </Engine>
      </Service>
    </Server>

3.2 修改默认配置目录:

.. code-block:: bash

    $ vim /data/tomcat7/bin/startup
    # 添加如下内容:
    #!/bin/bash
    #
    export CATALINA_BASE=/data/tomcat
    exec /opt/tomcat7/bin/catalina.sh "$@"

    $ chmod +x /data/tomcat/bin/startup

3.3 修改日志、PID目录:

.. code-block:: bash

    $ vim /data/tomcat7/bin/setenv.sh
    # 添加如下内容:
    #--============================================--#
    #                   环境相关
    #--============================================--#
    export JAVA_HOME="/usr/java/jdk1.7.0_55"
    export CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
    export CATALINA_PID="$CATALINA_BASE"/vars/run/tomcat7.pid
    export CATALINA_TMPDIR="$CATALINA_BASE"/vars/tmp
    
    
    #--============================================--#
    #                 JVM资源相关
    #--============================================--#
    CATALINA_OPTS="-server -Xmx400m -Xms400m
                   -XX:MaxPermSize=128m -XX:PermSize=128m
                   -XX:+UseParallelGC -XX:ParallelGCThreads=4"
    
    
    #--============================================--#
    #    开启JXM功能 (**注意修改hostname及端口**)
    #--============================================--#
    CATALINA_OPTS="$CATALINA_OPTS
     -Djava.rmi.server.hostname=VM01
     -Dcom.sun.management.jmxremote=true
     -Dcom.sun.management.jmxremote.port=18080
     -Dcom.sun.management.jmxremote.ssl=false
     -Dcom.sun.management.jmxremote.authenticate=false"
    
    
    #--============================================--#
    #                开启GC日志
    #--============================================--#
    CATALINA_OPTS="$CATALINA_OPTS
     -XX:+PrintGCDateStamps 
     -XX:+PrintGCDetails
     -Xloggc:${CATALINA_BASE}/logs/gc/gc.log"
    
    
    #--============================================--#
    #                开启HeapDump
    #--============================================--#
    CATALINA_OPTS="$CATALINA_OPTS
     -XX:+HeapDumpOnOutOfMemoryError 
     -XX:HeapDumpPath=${CATALINA_BASE}/logs/dump/heapdump.bin"

修改日志策略::

    $ sed -i '/^handlers =/ s/^/#/' /data/tomcat7/conf/logging.properties


四、启动程序
----------

4.1 启动应用程序::
    
二进制启动::

    $ su tomcat -s /bin/bash -c "/data/tomcat7/bin/startup start"

SysV启动脚本::

    $

supervisor启动配置:

.. code-block:: bash
    [program:tomcat7]
    command=/data/tomcat/bin/startup run
    stdout_logfile=/data/tomcat/logs/catalina.out
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10
    redirect_stderr=true

.. warning::

    选择一种启动方式即可，一般使用SysV启动脚本启动即可。如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。

4.2 验证部署:

.. code-block:: bash
    
    $ curl http://127.0.0.1:8080
    Index Successful!

五、附属功能
------------
