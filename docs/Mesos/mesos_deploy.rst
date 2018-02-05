===================
[部署]-RPM部署Mesos
===================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: CentOS Linux release 7.3.1611 (Core)
    系统内核: Linux version 3.10.0-514.el7.x86_64
    部署类型: rpm
    操作用户: root
    运行用户: root
    软件版本: mesos-1.3.1
    功能描述: 基于mesos使用marathon调度docker容器服务。

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - Mesos-Master, Mesos-Slave, Marathon, Zookeeper
  * - VM01
    - 192.168.182.101
    - Mesos-Slave, Zookeeper
  * - VM01
    - 192.168.182.101
    - Mesos-Slave, Zookeeper

0.3 资源规划:
            
.. code-block:: bash

    # 目录规划说明
    部署位置: rpm默认
    配置目录: rpm默认
    数据目录: rpm默认
    日志目录: rpm默认
    运行目录: rpm默认

.. code-block:: bash

    # 端口规划说明
    5050: (TCP-HTTP) Mesos   Web管理页面端口。
    8080: (TCP-HTTP) Maraton Web管理页面端口。

0.4 依赖资源:

.. code-block:: bash

    # 依赖服务说明    
    Docker Service    : version 1.12.6 (yum install docker)
    Zookeeper Service : version 3.4.9  (tar方式部署，根据相关部署文档。)

    # 依赖部署文件
    根据yum自动安装
    
    # 相关地址连接
    官方文档: http://mesos.apache.org/documentation

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正确部署，蓝色为说明提示。


一、解决依赖
------------

1.1 安装部署依赖:

.. code-block:: bash

    # 安装配置 Mesosphere 仓库
    rpm -ivh http://repos.mesosphere.com/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm

1.2 安装运行依赖:

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


1.3 创建运行用户::

    # 使用rpm默认用户

1.3 修改系统配置:

.. code-block:: bash

    # 修改hosts文件。如果你使用了内部DNS服务器则不需要修改此文件，否则请按照操作实行。
    vim /etc/hosts

    ↓ ↓ ↓ ↓ ↓ 添加如下内容 ↓ ↓ ↓ ↓ ↓
    192.168.182.101 VM01
    192.168.182.102 VM02
    192.168.182.103 VM03
    
.. code-block:: bash

    # 配置时间周期同步任务。
    crontab -e

    ↓ ↓ ↓ ↓ ↓ 添加如下内容 ↓ ↓ ↓ ↓ ↓
    # 每两小时 Linux 系统就会自动的进行网络时间校准
    00 */2 * * * root /usr/sbin/ntpdate cn.pool.ntp.org

.. code-block:: bash

    # 修改资源限制，打开文件数限制。
    vim /etc/security/limits.d/90-nofile.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    root          soft    nofile     65535
    root          hard    nofile     65535


    # 修改资源限制，打开进程数限制。
    vim /etc/security/limits.d/90-nproc.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    root          soft    nproc     unlimited
    root          hard    nproc     unlimited


二、安装程序
------------

2.1 安装所需组件:

.. code-block:: bash

    # 安装所需组件包 【Master,Slave节点】
    yum -y install mesos 

.. code-block:: bash

    # 安装所需组件包 【Marathon节点】
    yum -y install marathon

2.2 整理程序目录::

    # 无需操作

2.3 创建所需文件::
    
    # 无需操作

2.4 修改文件权限::

    # 无需操作

2.5 添加登录描述:

.. code-block:: bash

    # 根据节点类型添加内容。【Master节点】
    cat >> /etc/motd << EOF
    -----=========== Mesos 服务 ===========-----
         节点类型: Master
         节点说明: Messos的Msster节点
    -----==================================-----
    EOF

.. code-block:: bash

    # 根据节点类型添加内容。【Slave节点】
    cat >> /etc/motd << EOF
    -----=========== Mesos 服务 ===========-----
         节点类型: Slave
         节点说明: Messos的Slave节点
    -----==================================-----
    EOF

.. code-block:: bash

    # 根据节点类型添加内容。【Marathon节点】
    cat >> /etc/motd << EOF
    -----=========== Marathon 服务 ===========-----
         节点类型: Marathon
         节点说明: Messos的框架服务
    -----=====================================-----
    EOF


三、修改配置
------------

3.1 配置文件结构:

.. code-block:: bash

    /etc/mesos
         |-- zk               # Mesos使用的Zookeeper地址，master及slave节点均使用此配置文件。
   
    /etc/mesos-master
         |-- hostname         # 主机名配置，此值会应用在页面链接上。如果管理机无法解析此地址请填写IP地址。
         |-- quorum           # 仲裁数量，用于多master时使用。
         |-- work_dir         # 此配置文件用于指定工作目录。

    /etc/mesos-slave
         |-- containerizers   # 此配置文件用于指定slave执行可使用的容器类型。
         |-- work_dir         # 此配置文件用于指定工作目录。
         
    /etc/maraton/conf
         |-- hostname         # 主机名配置，此值会应用在页面链接上。如果管理机无法解析此地址请填写IP地址。
         |-- master           # Mesos的Zookeeper地址，用户发现Master节点。
         |-- zk               # Marathon使用的Zookeeper地址。

3.2 编辑主要配置:

.. code-block:: bash

    # 为mesos添加zookeeper地址。【Master节点】
    echo "zk://192.168.182.101:2181,192.168.182.102:2182,192.168.182.103:2183/mesos" > /etc/mesos/zk

.. code-block:: bash

    # 添加slave执行容器种类。【Slave节点】
    echo "docker,mesos" >  /etc/mesos-slave/containerizers

.. code-block:: bash

    # 修改marathon相关配置。【Mrathon节点】
    echo "zk://192.168.182.101:2181,192.168.182.102:2182,192.168.182.103:2183/mesos" >  /etc//etc/maraton/conf/master
    echo "zk://192.168.182.101:2181,192.168.182.102:2182,192.168.182.103:2183/marathon" >  /etc//etc/maraton/conf/zk

.. note::

    配置文件使用规则。以参数名为文件名，参数值为文件内容即可。

3.3 修改默认目录::

    # 保持默认路径即可，默认路径为/var/lib/mesos

3.4 修改启动脚本:

.. code-block:: bash

    # 保持默认即可。路径: /usr/lib/systemd/system/mesos-slave.service
    [Unit]
    Description=Mesos Master
    After=network.target
    Wants=network.target

    [Service]
    ExecStart=/usr/bin/mesos-init-wrapper master
    Restart=always
    RestartSec=20
    LimitNOFILE=16384

    [Install]
    WantedBy=multi-user.target

.. code-block:: bash

    # 保持默认即可。路径: /usr/lib/systemd/system/mesos-slave.service
    [Unit]
    Description=Mesos Slave
    After=network.target
    Wants=network.target

    [Service]
    ExecStart=/usr/bin/mesos-init-wrapper slave
    KillMode=process
    Restart=always
    RestartSec=20
    LimitNOFILE=16384
    CPUAccounting=true
    MemoryAccounting=true

    [Install]
    WantedBy=multi-user.target


四、启动程序
------------

4.1 启动应用程序:
    
.. code-block:: bash

    # 启动master节点命令。
    systemctl start mesos-master

.. code-block:: bash

    # 启动slave节点命令。
    systemctl start mesos-slave


4.2 检测启动状态:

.. code-block:: bash
    
    # 检测master节点命令。
    system status mesos-master

.. code-block:: bash

    # 检测slave节点命令。
    system status mesos-slave


4.3 开机启动设置:

.. code-block:: bash

    # 查看开机启动状态
    systemctl list-unit-files --type=service | egrep "mesos|marathon"

.. code-block:: bash

    # master节点操作。关闭slave开机启动，打开master开机启动。
    systemctl disable mesos-slave
    systemctl enable  mesos-master

.. code-block:: bash

    # slave节点操作。关闭master开机启动，打开slave开机启动。
    systemctl disable mesos-master
    systemctl enable  mesos-slave

.. code-block:: bash

    # slave节点操作。关闭master开机启动，打开slave开机启动。
    systemctl enable marathon

4.4 启动之后操作::

    登录界面 http://192.168.182.101:5050 进行查看相关信息(Mesos管理页面)。   
    登录界面 http://192.168.182.101:8080 进行查看相关信息(Marathon管理页面)。   


五、附属功能
------------
