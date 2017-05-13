====================
[部署]-单机部署5.6.X
====================

1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    操作用户: root
    运行用户: mysql
    软件版本: mysql-5.6.36
    部署位置: /opt/mysql
    配置目录: /data/mysql/conf
    数据目录: /data/mysql/data
    日志目录: /data/mysql/logs
    临时目录: /data/mysql/temp
    PID 目录: /data/mysql/run
    所需文件: mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

..
   1.2 相关地址::
    下载地址
    ---
    智能安装: 
   1.3 关键命令::
    mysql mysqldump

2 解决依赖
----------

2.1 安装依赖组件::

    $ yum install libaio perl

2.1 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 3306 mysql

2.3 创建所需目录::

    $ mkdir -p /data/mysql && cd /data/mysql
    $ mkdir conf data logs temp run
    $ chown -R mysql:mysql /data/mysql
    $ chown -R root:root /data/mysql/conf

3 安装程序
----------

3.1 解压软件包::

    $ cd /tmp
    $ tar xf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz -C /opt
    $ ln -sv /opt/mysql-5.6.36-linux-glibc2.5-x86_64/ /opt/mysql

3.2 创建所需文件::

    $ cp support-files/my-default.cnf /data/mysql/conf/my.cnf
    $ ln -sv /data/mysql/conf/my.cnf /etc/my.cnf
    $ sed -i 's/\/usr\/local\/mysql/\/opt\/mysql/g' bin/mysqld_safe               # 修改脚本中程序路径
    $ sed -i 's/\/usr\/local\/mysql/\/opt\/mysql/g' support-files/mysql.server    # 修改脚本中程序路径
    $ cp support-files/mysql.server /etc/init.d/mysql

3.3 初始化数据库::

    $ cd /opt/mysql
    $ scripts/mysql_install_db --user=mysql --basedir=/opt/mysql --datadir=/data/mysql/data

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    $ vim /data/mysql/conf/my.cnf
    # 添加如下内容:
    [mysqld]
    basedir                = /usr/local/mysql
    datadir                = /data/mysql/data
    log_error              = /data/mysql/logs/mysql.err
    socket                 = /tmp/mysql.sock
    tmpdir                 = /data/mysql/temp
    symbolic-links         = 0
    max_connections        = 1000
    max_allowed_packet     = 512M
    character-set-server   = utf8
    lower_case_table_names = 1
    transaction_isolation  = READ-COMMITTED
    skip-name-resolve
    skip-external-locking
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

5 启动程序
----------

5.1 启动命令::
    
    $ cd /opt/mysql/bin
    $ setsid ./mysqld_safe --defaults-file=/data/mysql/conf/my.cnf &>/dev/null

5.2 SysV启动脚本::

    $ service mysql start

.. note::
    
    一般使用SysV启动脚本启动即可。

5.3 验证部署:

.. code-block:: bash

    $ mysqladmin -h 127.0.0.1 -p 3306 ping
    mysqld is alive

6 规范环境
----------

6.1 安全初始化::

    $ mysql -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'ylzone' WITH GRANT OPTION"
    $ mysql -e "DELETE FROM mysql.user WHERE host != '%'"
    $ mysql -e "FLUSH PRIVILEGES"
    $ mysql -u root -p ylzone                               # 连接测试

6.2 开机启动::

    $ chkconfig --add mysql
    $ chkconfig mysql on

6.3 添加PATH:

.. code-block:: bash

    $ vim /etc/profile.d/mysql.sh
    # 添加如下内容:
    PATH=$PATH:/opt/mysql/bin
    export PATH
    $ source /etc/profile.d/mysql.sh

.. note::

    如果不需要编译等相关操，操作到 ``6.3小节`` 即可。

6.4 添加includea::

    $ ln -sv /opt/mysql/include /usr/include/mysql

6.5 添加库文件::

    $ echo '/opt/mysql/lib' > /etc/ld.so.conf.d/mysql.conf
    $ ldconfig                                               # 让系统重新载入系统库

6.6 添加man帮助:

.. code-block:: bash
    
    $ vim /etc/man.config
    MANPATH /opt/mysql/man
