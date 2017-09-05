===============
[部署]-部署DRBD
===============

零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: root
    软件版本: drbd-8.4.9-1
    所需文件: kmod-drbd84-8.4.9-1.el6.elrepo.x86_64.rpm, drbd84-utils-8.9.8-1.el6.elrepo.x86_64.rpm

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - drbd84(初始Primary)
  * - VM02
    - 192.168.182.102
    - drbd84(初始Secondary)
    
0.3 目录说明::

    程序路径: /sbin/drbdadm,/sbin/drbdmeta,/sbin/drbdsetup
    配置目录: /etc/drbd.d
    启动脚本: /etc/init.d/drbd
    运行目录: /var/run/drbd

0.4 端口说明::

    7788: 处理数据端口

0.5 补充说明:

提示操作种类::

    每个操作步骤后红色框字体为操作提示，详细如下：

    所有节点: 需要在节点说明中规划的所有节点进行操作。
    指定节点: 需要在节点说明中规划的指定节点进行操作。
    环境相关: 此操作或配置和部署环境有关系，如主机名，IP，目录等。


    每个操作部署下方会出现带颜色提示框，详细如下：
    
    蓝色提示: 主要为提示、介绍、说明等信息。
    红色提示: 要慎重操作此步骤可能影响正常进度。

.. warning::

    注意如果操作步骤中出现提示，请仔细检查现场部署环境和文档中的差异，最终以现场环境为准。否则将无法完成部署。

运行产出说明::

    创建块设备: /dev/drbd0



..
    ---
    智能安装: 
   1.3 关键命令::
    mysql mysqldump

一、解决依赖
------------

1.1 安装软件源库 ``所有节点``::

    $ rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    $ rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm


二、安装程序
------------

2.1 安装所需软件::

    $ yum install drbd84 kmod-drbd84


2.6 设置开机启动::

    $ chkconfig drbd on
    $ chkconfig --list drbd


三、修改配置
------------

3.1 编辑配置文件 ``所有节点`` ``环境相关``:

.. code-block:: bash

    $ vim /etc/drbd.d/global_common.conf

    ↓ ↓ ↓ ↓ ↓ 修改如下内容 ↓ ↓ ↓ ↓ ↓
    global {
        usage-count no;
    }

    common {
        protocol C;                   # 选择的是drbd的C 协议（数据同步协议，C为收到数据并写入后返回，确认成功） 
        disk {
                on-io-error detach;   # 同步错误的做法是分离
        }

        net {
                rate 30M;             # 同步速率视带宽而定
        }

    }

.. code-block:: bash

    $ vim /etc/drbd.d/mysql-db.res

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    resource mysql-db {                        # 创建一个资源，名字叫"mysql-db" 
        on VM01 {                              # 设定一个节点，分别以各自的主机名命名 
                device /dev/drbd0;             # 设定资源设备/dev/drbd0 指向实际的物理分区 /dev/sdb
                disk /dev/sdb;
                address 192.168.182.101:7788;  # 设定监听地址以及端口 
                meta-disk internal;            # internal表示是在同一个局域网内 
        }

        on VM02 {
                device /dev/drbd0;
                disk /dev/sdb;
                address 192.168.182.102:7788;
                meta-disk internal;            
        }
    }


四、启动程序
------------

4.1 启动之前操作 ``所有节点``::

    $ drbdadm create-md mysql-db

4.2 启动应用程序 ``所有节点``::

    $ service drbd start

4.3 检测启动状态 ``所有节点``:

.. code-block:: bash

    # 检测方式一
    $ service drbd status
    drbd driver loaded OK; device status:
    version: 8.4.9-1 (api:1/proto:86-101)
    GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
    m:res       cs         ro                   ds                     p  mounted  fstype
    0:mysql-db  Connected  Secondary/Secondary  Diskless/Inconsistent  C


.. code-block:: bash

    # 检测方式二
    $ drbd-overview
     0:mysql-db/0  Connected Secondary/Secondary Diskless/Inconsistent 

.. code-block:: bash

    # 检测方式三
    $ cat /proc/drbd
    version: 8.4.9-1 (api:1/proto:86-101)
    GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
     0: cs:Connected ro:Secondary/Secondary ds:Diskless/Inconsistent C r-----
         ns:0 nr:0 dw:0 dr:0 al:0 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:b oos:0


4.4 启动后续操作 ``Primary节点``:

.. code-block:: bash

    # 指定Primary节点
    $ drbdadm -- --overwrite-data-of-peer primary mysql-db

    # 检测执行状态
    $ service drbd status
    drbd driver loaded OK; device status:
    version: 8.4.9-1 (api:1/proto:86-101)
    GIT-hash: 9976da086367a2476503ef7f6b13d4567327a280 build by mockbuild@Build64R6, 2016-12-13 18:38:15
    m:res       cs         ro                 ds                 p  mounted  fstype
    0:mysql-db  Connected  Primary/Secondary  UpToDate/Diskless  C  /data    ext4

.. code-block:: bash

    # 格式化并挂载
    $ mkfs.ext4 /dev/drbd0
    $ mkdir /data
    $ mount /dev/drbd0 /data/


五、附属功能
------------

5.1 主动切换Primary/Secondary:

.. code-block:: bash

    # Primary节点操作，Primary到Secondary。
    $ umount /data
    $ drbdadm secondary mysql-db

    # Secondary节点操作，Secondary到Primary。
    $ drbdadm  primary mysql-db
    $ mount /dev/drbd0 /data

.. note::

    切换操作后可以使用 ``service drbd status`` 来查看执行结果。

5.2 脑裂后的处理:

.. code-block:: bash
    
    # 当出现脑裂后，会导致drbd两边的磁盘数据不一致，在确定要作为从的节点上切换成secondary，并放弃该资源的数据。
    $ drbdadm secondary mysql-db
    $ drbdadm -- --discard-my-data connect mysql-db

    # 在要作为primary的节点重新连接secondary（如果这个节点当前的连接状态为WFConnection的话，可以省略）。
    $ drbdadm connect mysql-db
