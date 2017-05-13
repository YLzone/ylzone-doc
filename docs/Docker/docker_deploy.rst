===================
[部署]-CenOS6.X部署
===================

1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: rpm部署
    操作用户: root
    运行用户: root
    软件版本: docker-1.7.1
    配置文件: /etc/sysconfig/docker、docker-storage、docker-network
    数据目录: /var/lib/docker
    日志文件: /var/log/docker
    所需文件: docker-io-1.7.1-2.el6.x86_64、kernel-lt-3.10.105-1.el6.elrepo.x86_64

..
   1.2 相关地址::
    下载地址
    ---
    智能安装: 
   1.3 关键命令::
    mysql mysqldump

2 解决依赖
----------

2.1 升级内核为3.10.X::

    $ rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
    $ yum install --enablerepo=elrepo-kernel install kernel-lt -y

    # 修改引导为新内核
    $ vim /boot/grub/grub.conf
    default=0

    # 重启查看内核版本
    # cat /proc/version 
    Linux version 3.10.105-1.el6.elrepo.x86_64 ......

.. note::
    
    注意 ``default`` 的值为你新安装内核的索引,根据你的具体情况修改，一般为0

2.2 安装epel源::

    $ yum install epel-release -y

3 安装程序
----------

3.1 使用yum安装docker::

    $ yum install docker-io

.. note::
    
    centos7 安装包是 ``docker`` 而不是 ``docker-io``

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    # 默认即可

5 启动程序
----------

5.1 SysV启动脚本::

    $ service docker start

5.3 验证部署:

.. code-block:: bash

    $ docker version
    Client version: 1.7.1
    Client API version: 1.19
    Go version (client): go1.4.2
    Git commit (client): 786b29d/1.7.1
    OS/Arch (client): linux/amd64
    Server version: 1.7.1
    Server API version: 1.19
    Go version (server): go1.4.2
    Git commit (server): 786b29d/1.7.1
    OS/Arch (server): linux/amd64

6 规范环境
----------

6.1 开机启动::

     chkconfig docker on
