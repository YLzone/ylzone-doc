==================
[部署]-部署Jenkins
==================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-696.el6.x86_64
    部署方式: tar
    操作用户: root
    运行用户: jenkins
    软件版本: Jenkins-2.60.3
    所需文件: Jenkins-2.60.3.war;
             jdk-8u60-linux-x64.gz;
             apache-tomcat-7.0.72.tar.gz

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - Jenkins
    
0.3 目录说明::

    -继承tomcat的目录

0.4 端口说明::

    8080: HTTP协议，提供服务所用。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进度，蓝色为说明提示。

0.5.1 相关资源::

    官方文档: https://jenkins.io/doc/
    官方下载: https://jenkins.io/download/


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

.. warning::

    Jenkins依赖jdk-8，不能使用jdk-7否则会报错。

1.2 创建运行用户::

    useradd -M -s /sbin/nologin -u 8080 jenkins

1.3 安装依赖组件::

    按照 Tomcat部署文档 进行部署 

二、安装程序
------------

2.1 解压缩软件包:

.. code-block:: bash

    cd /tmp
    cp Jenkins-2.60.3.war /data/tomcat7/apps/jenkins


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: bash

    暂无


四、启动程序
------------

4.1 启动应用程序::
    
    /data/tomcat7/sbin/startup start

4.2 检测启动状态::

    -

4.3 启动后续操作:


五、附属功能
------------

5.1 相关模块介绍::

    Ant Plugin                        提供在Jenkins中使用ant脚本。
    Email Extension Plugin            提供发送HTML格式的邮件。
    Credentials Plugin                保存Jenkins用到的所有凭证。
    Subversion Plugin                 增加对svn(SVKKit)的支持。
    Parameterized Trigger Plugin      可以让你在多个项目中传递参数。
    Workspace Cleanup Plugin          这个插件可以在每次build后清空工作目录。
    Publish Over SSh                  通过ssh发布建立好的构建。
    SSH plugin                        在构建前后构建后在远端主机上执行脚本。
    Deploy to container Plugin        这个插件支持war/ear文件部署。
    External Moniter Job Type Plugin  外部检测工作类型的插件。
    Windows Slaves Plugin             这个插件提供链接控制windows节点的功能。
