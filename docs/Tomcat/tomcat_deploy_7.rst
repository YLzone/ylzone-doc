=================
[部署]-部署tomcat
=================


零、背景介绍
------------

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

    部署位置: /program/service/tomcat7
    配置目录: /data/project/tomcat7/conf
    数据目录: /data/project/tomcat7/data
    日志目录: /data/project/tomcat7/logs
    运行目录: /data/project/tomcat7/vars

0.4 端口说明::

    8080: 提供http服务的端口。
    18080: 提供JMX服务的端口。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正确部署，蓝色为说明提示。


一、解决依赖
------------

1.1 安装依赖组件::

安装JDK:

.. code-block:: bash

    # 安装软件包
    $ cd ~/packages
    $ mkdir -pv /program/basic/java
    $ tar xvf jdk-8u60-linux-x64.gz -C /program/basic/java
    $ ln -sv /program/basic/java/jdk1.8.0_60 /program/basic/java/latest
    $ ln -sv /program/basic/java/latest /program/basic/java/default
    $ chown -R depuser:depuser /usr/java/jdk1.8.0_60
    $ echo 'export JAVA_HOME=/program/basic/java/default' > /etc/profile.d/java.sh
    $ echo 'export PATH=${PATH}:${JAVA_HOME}/bin' >> /etc/profile.d/java.sh
    $ source /etc/profile.d/java.sh

    # 验证安装，显示如下内容表示成功。
    $ java -version
    java version "1.8.0_60"
    Java(TM) SE Runtime Environment (build 1.7.0_55-b13)
    Java HotSpot(TM) 64-Bit Server VM (build 24.55-b03, mixed mode)

1.2 创建运行用户::

    $ useradd -u 8080 appuser
    $ useradd admuser && useradd loguser 

.. note::

    为了便于管理和安全考虑，用户可分为 ``管理用户:admuser`` ``运行用户:appuser`` ``日志用户loguser`` 三类。

1.3 修改hosts文件:

.. code-block:: bash

    $ vim /etc/hosts
    # 添加如下内容
    192.168.182.101 VM01
    
.. note::

    如果你使用了内部DNS服务器则不需要修改此文件，否则请按照操作实行。

1.4 配置时间同步 ``所有节点``:

.. code-block:: bash

    $ crontab -e

    ↓ ↓ ↓ ↓ ↓ 添加如下内容 ↓ ↓ ↓ ↓ ↓
    # 每两小时 Linux 系统就会自动的进行网络时间校准
    00 */2 * * * root /usr/sbin/ntpdate cn.pool.ntp.org

1.5 修改资源限制 ``所有节点``:

.. code-block:: bash

    $ vim /etc/security/limits.d/90-nofile.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    tomcat          soft    nofile     65535
    tomcat          hard    nofile     65535

    $ vim /etc/security/limits.d/90-nproc.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    tomcat          soft    nproc     unlimited
    tomcat          hard    nproc     unlimited


二、安装程序
------------

2.1 解压软件包::

    $ cd ~/packages
    $ tar xf apache-tomcat-7.0.82.tar.gz -C /program/service/tomcat
    $ mv /program/service/tomcat/apache-tomcat-7.0.82 /program/service/tomcat/tomcat-7.0.82

2.2 整理文件::

    $ mv /program/service/tomcat/tomcat-7.0.82/{conf,conf.orig}
    $ rm -fv /program/service/tomcat/tomcat-7.0.82/{LICENSE,NOTICE,RELEASE-NOTES,RUNNING.txt}
    $ rm -fv /program/service/tomcat/tomcat-7.0.82/bin/*.bat
    $ rm -rfv /program/service/tomcat/tomcat-7.0.82/{logs,temp,webapps,work}

2.3 创建所需目录::

    $ mkdir -pv /data/project/tomcat7/{apps,sbin,conf,data,logs,vars,back}
    $ mkdir -pv /data/project/tomcat7/vars/{run,tmp,wap,wrk}
    $ mkdir -pv /data/project/tomcat7/vars/wap/ROOT
    
2.4 创建所需文件::
    
    $ cp /program/service/tomcat/tomcat-7.0.82/conf.orig/* /data/project/tomcat7/conf
    $ echo 'Index Successful!' > /data/project/tomcat7/vars/wap/ROOT/index.html
    $ touch /data/project/tomcat7/sbin/startup
    $ touch /data/project/tomcat7/sbin/setenv.sh

2.5 修改文件权限::

    $ chown -R root:root /program/service/tomcat/tomcat-7.0.82
    $ chown -R appuser:appuser /data/project/tomcat7

2.6 设置开机启动::

    $ sed -i '6i su tomcat -s /bin/bash -c "/opt/tomcat7/bin/startup start"' /etc/rc.d/rc.local

.. warning::

    如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: xml

    $ vim /data/project/tomcat7/conf/server.xml

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
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
    
          <!-- No.2 default root is CATALINA_BASE of appBase -->
          <Host name="localhost"  appBase="apps" 
                unpackWARs="true" autoDeploy="true" workDir="vars/wrk">

            <!-- No.1 default root is appBase of docBase -->
            <!-- Context path="/apps" docBase="../../apps" reloadable="flase"/ -->

            <!-- No.3 default root is appBase of docBase for ROOT -->
            <Context path="" docBase="../vars/wap/ROOT"/>

            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                   prefix="localhost_access_log." suffix=".txt"
                   pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    
          </Host>
        </Engine>
      </Service>
    </Server>

3.2 修改默认配置目录:

.. code-block:: bash
 
    $ vim /data/tomcat7/sbin/startup

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    #!/bin/bash
    #
    SELF_BASE=$(dirname $(dirname $(readlink -f $0)))

    export CATALINA_BASE=${SELF_BASE}
    source ${SELF_BASE}/sbin/setenv.sh
    exec /program/service/tomcat/tomcat-7.0.82/bin/catalina.sh "$@"

.. code-block:: bash

    # 赋予脚本执行权限
    $ chmod +x /data/project/tomcat7/sbin/startup

3.3 修改日志、PID目录:

.. code-block:: bash

    $ vim /data/project/tomcat7/sbin/setenv.sh

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    #--============================================--#
    #                   环境相关
    #--============================================--#
    export JAVA_HOME="/program/basic/java/default"
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

    $ sed -i '/^handlers =/ s/^/#/' /data/project/tomcat7/conf/logging.properties
    $ sed -i '18s/.handlers/handlers/' /data/project/tomcat7/conf/logging.properties


四、启动程序
------------

4.1 启动应用程序::
    
二进制启动::

    $ su appuser -s /bin/bash -c "/data/project/tomcat7/sbin/startup start"

.. note::

    如果启动过程较慢，可能是由于系统随机数熵不够导致。查看 ``/proc/sys/kernel/random/entropy_avail`` 获取该值。可以安装 ``yum install haveged`` 增大该值。 

SysV启动脚本::

    -

supervisor启动配置:

.. code-block:: bash

    [program:tomcat7]
    command=/data/project/tomcat7/sbin/startup run
    stdout_logfile=/data/project/tomcat7/logs/supervisor.out
    stdout_logfile_maxbytes=500MB
    stdout_logfile_backups=10
    redirect_stderr=true

.. warning::

    选择一种启动方式即可，一般使用SysV启动脚本启动即可。如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。

4.2 验证部署:

.. code-block:: bash
    
    # 测试主页
    $ curl http://127.0.0.1:8080
    Index Successful!


五、附属功能
------------

5.1 配置使用Redis做session共享:

.. code-block:: bash

    # 拷贝jar包到tomcat的lib目录
    $ cp /tmp/tomcat/resource/jedis-2.5.2.jar \
                              commons-pool2-2.2.jar \
                              tomcat-redis-session-manage-tomcat7.jar \
         /opt/tomcat7/lib

.. code-block:: bash
    
    # 修改配置文件
    $ vim /data/tomcat7/conf/context.xml

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    <?xml version='1.0' encoding='utf-8'?>

    <!-- allowLinking="true" 可以使用软连接访问目录 -->
    <Context allowLinking="true">

        <!-- Default set of monitored resources -->
        <WatchedResource>WEB-INF/web.xml</WatchedResource>

        <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />        
        <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
            host="SES-RDS01.HJ.BJ.JRX"
            port="6379"
            database="0" />

            <!-- host="localhost"             Redis地址 -->
            <!-- port="6379"                  Redis端口 -->
            <!-- password="123456"            Redis密码 -->
            <!-- database="0"                 存储Session的Redis库编号 -->
            <!-- maxInactiveInterval="60"     Session失效的间隔（秒） -->

    </Context>

.. code-block:: bash
    
    # 添加session测试页面，使用浏览器访问。
    $ vim /data/tomcat7/apps/session.jsp

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    <%@ page contentType="text/html; charset=UTF-8" %>
    <%@ page import="java.util.*" %>
    <html><head><title>Cluster App Test</title></head>
    <body>
    Server Info:
    <%
    out.println(request.getLocalAddr() + " : " + request.getLocalPort()+"<br>");%>
    <%
      out.println("<br> ID " + session.getId()+"<br>");
      // 如果有新的 Session 属性设置
      String dataName = request.getParameter("dataName");
      if (dataName != null && dataName.length() > 0) {
         String dataValue = request.getParameter("dataValue");
         session.setAttribute(dataName, dataValue);
      }
      out.print("<b>Session 列表</b>");
      Enumeration e = session.getAttributeNames();
      while (e.hasMoreElements()) {
         String name = (String)e.nextElement();
         String value = session.getAttribute(name).toString();
         out.println( name + " = " + value+"<br>");
             System.out.println( name + " = " + value);
       }
    %>
      <form action="session.jsp" method="POST">
        名称:<input type=text size=20 name="dataName">
         <br>
        取值:<input type=text size=20 name="dataValue">
         <br>
        <input type=submit>
       </form>
    </body>
    </html>
