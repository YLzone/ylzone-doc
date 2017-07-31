====================
[部署]-单机部署5.6.X
====================

零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    部署方式: tar
    操作用户: root
    运行用户: mysql
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
    - mysql
    
0.3 目录说明::

    程序目录: /opt/mysql
    配置目录: /data/mysql/conf
    数据目录: /data/mysql/data
    日志目录: /data/mysql/logs
    临时目录: /data/mysql/var/tmp
    PID 目录: /data/mysql/var/run

0.4 端口说明::

    3306: 默认端口，提供服务所用。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进入，蓝色为说明提示。

..
   1.2 相关地址::
    下载地址
    ---
    智能安装: 
   1.3 关键命令::
    mysql mysqldump

一、解决依赖
------------

1.1 安装依赖组件::

    $ yum install libaio perl

1.2 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 3306 mysql


二、安装程序
------------

2.1 解压缩软件包::

    $ cd /tmp
    $ tar xf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz -C /opt
    $ ln -sv /opt/mysql-5.6.36-linux-glibc2.5-x86_64/ /opt/mysql

2.2 创建所需目录::

    $ mkdir -pv /data/mysql/{conf,data,logs,vars}
    $ mkdir -pv /data/mysql/var/{run,tmp}

2.3 创建所需文件::

    $ cp support-files/my-default.cnf /data/mysql/conf/my.cnf
    $ ln -sv /data/mysql/conf/my.cnf /etc/my.cnf
    $ sed -i 's/\/usr\/local\/mysql/\/opt\/mysql/g' bin/mysqld_safe               # 修改脚本中程序路径
    $ sed -i 's/\/usr\/local\/mysql/\/opt\/mysql/g' support-files/mysql.server    # 修改脚本中程序路径
    $ cp support-files/mysql.server /etc/init.d/mysql

2.4 修改文件权限::

    $ chown -R root:root /opt/mysql
    $ chown -R mysql:mysql /data/mysql
    $ chown -R root:root /data/mysql/conf
    
2.5 修改环境变量:

.. code-block:: bash

    $ vim /etc/profile.d/mysql.sh
    # 添加如下内容:
    PATH=$PATH:/opt/mysql/bin
    export PATH

    $ source /etc/profile.d/mysql.sh

2.6 设置开机启动::

    $ chkconfig --add mysql
    $ chkconfig mysql on

.. warning::

    如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。

2.7 初始化数据库::

    $ cd /opt/mysql
    $ scripts/mysql_install_db --user=mysql --basedir=/opt/mysql --datadir=/data/mysql/data

三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: bash

    $ vim /data/mysql/conf/my.cnf
    # 添加如下内容:
    [mysqld]
    user                   = mysql
    port                   = 3306
    basedir                = /opt/mysql
    datadir                = /data/mysql/data
    log_error              = /data/mysql/logs/mysql.err
    socket                 = /tmp/mysql.sock
    tmpdir                 = /data/mysql/vars/tmp
    pid_file               = /data/mysql/vars/run/mysqld.pid
    symbolic-links         = 0
    max_connections        = 1000
    max_allowed_packet     = 512M
    character-set-server   = utf8
    lower_case_table_names = 1
    transaction_isolation  = READ-COMMITTED
    skip-name-resolve
    skip-external-locking
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

四、启动程序
------------

4.1 启动应用程序:
    
二进制启动::

    $ cd /opt/mysql/bin
    $ setsid ./mysqld_safe --defaults-file=/data/mysql/conf/my.cnf &>/dev/null

SysV启动脚本::

    $ service mysql start

supervisor启动配置:

.. code-block:: bash

    [program:mysql]
    command=/usr/local/python2.7.9/bin/pidproxy /data/mysql/data/mysqld.pid
     /opt/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf
    stdout_logfile=/tmp/mysql.log
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10

.. note::
    
    选择一种启动方式即可，一般使用SysV启动脚本启动即可。

4.2 检测启动状态::

    $ mysqladmin -h 127.0.0.1 -p 3306 ping
    mysqld is alive         # 返回此结果运行正常           

4.3 启动后续操作:

安全初始化root账号::

    $ mysql -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'ylzone' WITH GRANT OPTION"
    $ mysql -e "DELETE FROM mysql.user WHERE host != '%'"
    $ mysql -e "FLUSH PRIVILEGES"
    $ mysql -u root -p ylzone                               # 连接测试

.. note::

    如果上述如步骤均操作正常，则mysql部署完成。酌情把相关地址、账号密码发送给使用者。

五、附属功能
------------

5.1 环境规范操作

添加include支持::

    $ ln -sv /opt/mysql/include /usr/include/mysql

添加lib支持::

    $ echo '/opt/mysql/lib' > /etc/ld.so.conf.d/mysql.conf
    $ ldconfig                                               # 让系统重新载入系统库

添加man帮助:

.. code-block:: bash
    
    $ vim /etc/man.config
    MANPATH /opt/mysql/man
    
.. note::

   ``5.1步骤`` 主要为支持编译等相关操，如无相关需要可忽略此步骤。

..
   添加管理用户进行对 mysql的管理
   如：添加admin或super用户，之后在sudoer中加入可操作mysql相关命令
