====================
[部署]-单机部署5.6.X
====================

零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.9
    系统内核: Linux version 2.6.32-696.el6.x86_64
    部署方式: tar
    操作用户: root
    运行用户: -
    软件版本: mysql-5.6.36
    所需文件: mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - maven
    
0.3 目录说明::

    程序目录: /opt/mysql
    备份目录: /data/mysql/back
    配置目录: /data/mysql/conf
    数据目录: /data/mysql/data
    日志目录: /data/mysql/logs
    临时目录: /data/mysql/vars/tmp
    运行目录: /data/mysql/vars/run

0.4 程序说明::

    程序类型: java应用
    依赖组件: jvm
    官方文档:
    官方下载: 

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进度，蓝色为说明提示。


一、解决依赖
------------

1.1 安装依赖组件:

.. code-block:: bash

    # 安装 Java Development Kit
    mkdir -v /usr/java
    tar xvf jdk-8u60-linux-x64.gz -C /usr/java
    ln -sv /usr/java/jdk1.8.0_60 /usr/java/latest
    ln -sv /usr/java/latest /usr/java/default
    chown -R root:root /usr/java/jdk1.8.0_60
    echo 'export JAVA_HOME=/usr/java/default' > /etc/profile.d/java.sh
    echo 'export PATH=${PATH}:${JAVA_HOME}/bin' >> /etc/profile.d/java.sh
    source /etc/profile.d/java.sh

    # 验证安装，显示如下内容表示成功。
    java -version


二、安装程序
------------

2.1 解压缩软件包::

    cd /tmp
    tar xf apache-maven-3.5.0-bin.tar.gz -C /opt
    mv /opt/apache-maven-3.5.0 /opt/maven
    echo "version: maven-3.5.0" >> /opt/maven/VERSION.md

2.4 修改文件权限::

    chown -R root:root /opt/maven
    
2.5 修改环境变量::

    echo 'export MAVEN_HOME=/opt/maven' > /etc/profile.d/maven.sh
    echo 'export PATH=${PATH}:${MAVEN_HOME}/bin' >> /etc/profile.d/maven.sh
    source /etc/profile.d/maven.sh


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: bash


四、启动程序
------------

4.1 检测安装状态::

    mvn -v


五、附属功能
------------

