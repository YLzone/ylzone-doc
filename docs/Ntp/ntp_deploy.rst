==============
[部署]-部署ntp
==============


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: ntp
    软件版本: 4.2.6p5-10.el6
    部署位置: rpm默认
    配置目录: /etc/ntp.conf
    数据目录: 
    日志目录: 
    PID 目录: 
    所需文件: ntp-4.2.6p5-10.el6.centos.2.x86_64

1.4 端口介绍::

    UDP:123       # 表示和客户端通信的接口

1.5 相关程序::

    nptd          # NTP 的主要 daemon 提供 NTP 服务，配置文件为 /etc/ntp.conf。 
    ntpq          # 可以列出目前我们的 NTP 与相关的上层 NTP 的状态。
    ntpstat       # 这个指令可以列出我们的 NTP 服务器的连接状态。
    ntpdata       # NTP 的 Client 端程序，用于连接服务器同步时间使用。


2 解决依赖
----------

2.1 创建所需目录::

    $ mkdir -p /data/ntp && cd /data/ntp
    $ mkdir conf data logs run

3 安装程序
----------

3.1 使用yum安装程序::

    $ yum install ntp -y

3.2 重定向配置文件::

    $ mv /etc/ntp.conf /data/ntp/conf/ntp.conf
    $ ln -sv /data/ntp/conf/ntp.conf /etc/ntp.conf

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    $ vim /etc/ntp.conf
    # 添加如下内容:
    # 1. 权限相关配置∶
    restrict default nomodify
    restrict 127.0.0.1

    # 2. 设定主机来源:
    server 0.centos.pool.ntp.org iburst
    server 1.centos.pool.ntp.org iburst
    server 2.centos.pool.ntp.org iburst
    server 3.centos.pool.ntp.org iburst

    # 3. 让NTP Server和其自身保持同步，如果定义的server都不可用时，将使用local时间作为ntp服务提供给ntp客户端:
    server  127.127.1.0
    fudge   127.127.1.0 stratum 10

    # 4. 运行中生成的数据文件:
    driftfile /var/lib/ntp/drift    # 时间差异分析
    pidfile   /var/run/ntpd.pid     # pid文件
    logfile   /var/log/ntp.log      # 日志文件

5 运行程序
----------

5.1 启动命令::
    
    $ service ntpd start


5.3 验证部署: 

.. code-block:: bash
    
    $ ntpq -pn
    remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    *173.255.246.13  207.224.49.219   2 u  208   64  370  167.860   52.316  47.432
    +163.172.177.158 5.103.128.88     3 u   74   64  376  300.796   31.106  35.150
    +212.47.249.141  5.103.128.88     3 u    3   64  373  301.198   37.867  29.863
     127.127.1.0     .LOCL.          10 l  678   64    0    0.000    0.000   0.000

.. code-block:: bash

    $ ntpstat
    synchronised to NTP server (173.255.246.13) at stratum 3 
       time correct to within 243 ms
       polling server every 64 s

.. code-block:: bash

    # 客户端执行
    $ ntpdate ip


6 规范环境
----------

6.1 开机启动::

    chkconfig ntpd on
    
6.2 日志轮转::

    log

7 补充说明
----------

7.1 主要配置说明:
