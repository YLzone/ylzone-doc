==================
[部署]-部署ansible
==================


1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: ntp
    软件版本: ansible-2.3.0.0-3.el6
    部署位置: rpm默认
    配置目录: /etc/ansible --> /data/ansible/conf
    数据目录: 无
    日志目录: 无
    PID 目录: 无
    所需文件: ansible-2.3.0.0-3.el6.noarch

1.2 配置结构::

    /etc/ansible/ansible.cfg    # 主配置文件
    /etc/ansible/hosts          # 主机配置文件
    /etc/ansible/roles          # 用于层次性，结构化地组织playbook，roles能够根据层次型结构自动自动装在变量文件、tasks以及handlers等。

1.3 环境描述::

    192.168.182.101    VM01    ansible
    192.168.182.102    VM02
    192.168.182.103    VM03

2 解决依赖
----------

2.1 安装epel软件源::

    $ yum install epel-release

2.2 升级ssh客户端版本

    $ yum upgread openssh-clients

.. note::

    在某些版本中，升级ssh客户端版本可以加快ansible的执行速度。

2.3 创建所需目录::

    $ mkdir -p /data/ansible && cd /data/ansible
    $ mkdir conf data logs

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

    $ vim /etc/ansible/

5 运行程序
----------

5.1 验证部署: 

.. code-block:: bash
    
    $ ntpq -pn


6 规范环境
----------

6.1 开机启动::

    chkconfig ntpd on
    
6.2 日志轮转::

    log

7 补充说明
----------

7.1 主要配置说明:
