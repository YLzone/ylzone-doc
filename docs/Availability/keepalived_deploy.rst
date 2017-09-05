=====================
[部署]-部署Keepalived
=====================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.9
    系统内核: Linux version 2.6.32-696.el6.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: root
    软件版本: keepalived-1.2.13
    所需文件: keepalived-1.2.13-5.el6_6.x86_64.rpm
             lm_sensors-libs-3.1.1-17.el6.x86_64.rpm
             net-snmp-libs-5.5-57.el6_8.1.x86_64.rpm

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - keepalived(初始Master)
  * - VM02
    - 192.168.182.102
    - keepalived(初始Slave)
    
0.3 目录说明::

    程序路径: /usr/sbin/keepalived
    配置目录: /etc/keepalived
    启动脚本: /etc/init.d/keepalived
    运行文件: /var/run/keepalived.pid, vrrp.pid, checkers.pid

0.4 端口说明::

    -

0.5 补充说明:


提示操作种类::

    每个操作步骤后红色框字体为操作提示，详细如下：

    所有节点: 需要在节点说明中规划的所有节点进行操作。
    指定节点: 需要在节点说明中规划的指定节点进行操作。
    环境相关: 此操作或配置和部署环境有关系，如主机名，IP，目录等。


    每个操作部署下方会出现带颜色提示框，详细如下：
    
    蓝色提示: 主要为提示、介绍、说明等信息。
    红色提示: 要慎重操作此步骤可能影响正常进度。

keepalived配置提示::

    keepalived 检测脚本周期为3秒，超时时间为3秒，连续错误次数为3次。
    keepalived 在配置过程中，参数的顺序很重要影响执行结果，配置后应使用tcpdump进行验证。

.. warning::

    注意如果操作步骤中出现提示，请仔细检查现场部署环境和文档中的差异，最终以现场环境为准。否则将无法完成部署。


一、解决依赖
------------

1.1 安装软件源库 ``所有节点``::

    $ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    $ rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm


二、安装程序
------------

2.1 安装所需软件 ``所有节点``::

    $ yum install keepalived


2.6 设置开机启动 ``所有节点``::

    $ chkconfig keepalived on
    $ chkconfig --list keepalived


三、修改配置
------------

3.1 编辑配置文件:


Master节点配置 ``Master节点`` ``环境相关``:

.. code-block:: bash

    $ vim /etc/keepalived/keepalived.conf

    ↓ ↓ ↓ ↓ ↓ 修改如下内容 ↓ ↓ ↓ ↓ ↓
    ! Configuration File for keepalived

    global_defs {              # 全局配置段，可配置邮件地址，标识等。
       router_id VM01          # 实例标识，默认为主机名。
    }
    vrrp_script chk_nginx_port {
        script "</dev/tcp/127.0.0.1/80"
        interval 3             # 调用脚本两次之间的间隔，默认为1秒。
        timeout 3              # 调用脚本的超时时间。
        fall 3                 # 脚本返回非0连续次数，满足后执行权重操作，不到此值恢复0重新计数。
        weight -10             # 权重值默认是2，取值范围-255到255。
                               # 当脚本执行码为0且权重大于0时，vrrp实例优先级增加；
                               # 当脚本执行码为非0且权重小于0时，vrrp实例优先级减小，其他情况优先级不变。
    }
    
    vrrp_instance VI_1 {
        state MASTER           # 初始状态，MASTER|BACKUP，其他机器加入举行选举，最高优先级的成为MASTER。
        interface eth0         # 指定该实例用户vrrp的网卡，用于发送vrrp。
        virtual_router_id 101  # 用于区分运行在相同NIC(和相同套接字)上的vrrpd的多个实例。
        priority 100           # 初始优先权，运行会根据权重增减。
        advert_int 1           # VRRP广告的时间间隔。
        authentication {       # 认证相关，避免非集群内成员干扰正常运行。
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {    # 虚拟IP地址(VIP)，此IP地址会在主机间漂移。
            192.168.200.16
        }
    }

Slave节点配置 ``Slave节点`` ``环境相关``:

.. code-block:: bash

    $ vim /etc/keepalived/keepalived.conf

    ↓ ↓ ↓ ↓ ↓ 修改如下内容 ↓ ↓ ↓ ↓ ↓
    ! Configuration File for keepalived

    global_defs {              # 全局配置段，可配置邮件地址，标识等。
       router_id VM02          # 实例标识，默认为主机名。
    }
    vrrp_script chk_nginx_port {
        script "</dev/tcp/127.0.0.1/80"
        interval 3             # 调用脚本两次之间的间隔，默认为1秒。
        timeout 3              # 调用脚本的超时时间。
        fall 3                 # 脚本返回非0连续次数，满足后执行权重操作，不到此值恢复0重新计数。
        weight -10             # 权重值默认是2，取值范围-255到255。
                               # 当脚本执行码为0且权重大于0时，vrrp实例优先级增加；
                               # 当脚本执行码为非0且权重小于0时，vrrp实例优先级减小，其他情况优先级不变。
    }
    
    vrrp_instance VI_1 {
        state SLAVE            # 初始状态，MASTER|BACKUP，其他机器加入举行选举，最高优先级的成为MASTER。
        interface eth0         # 指定该实例用户vrrp的网卡，用于发送vrrp。
        virtual_router_id 101  # 用于区分运行在相同NIC(和相同套接字)上的vrrpd的多个实例。
        priority 99            # 初始优先权，运行会根据权重增减。
        advert_int 1           # VRRP广告的时间间隔。
        authentication {       # 认证相关，避免非集群内成员干扰正常运行。
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {    # 虚拟IP地址(VIP)，此IP地址会在主机间漂移。
            192.168.200.16
        }
    }


四、启动程序
------------


4.1 启动应用程序 ``所有节点``::

    $ service keepalived start

4.2 检测启动状态 ``所有节点``:

.. code-block:: bash

    # 检测方式一
    $ service keepalived status
    keepalived (pid  6461) is running...

.. code-block:: bash

    # 检测方式二
    $ tcpdump -nn -i any net 224.0.0.0/8

.. code-block:: bash

    # 检测方式三，检测VIP状态。
    $ ip addr show eth0
    2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:0c:29:95:6e:70 brd ff:ff:ff:ff:ff:ff
    inet 192.168.182.102/24 brd 192.168.182.255 scope global eth0
    inet 192.168.200.16/32 scope global eth0


五、附属功能
------------
