====================
[部署]-单机部署5.7.X
====================

1 背景介绍
----------

1.1 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    操作用户: root
    运行用户: mysql
    软件版本: mysql-5.7.16
    部署位置: /opt/mysql
    配置目录: /data/mysql/conf
    数据目录: /data/mysql/data
    日志目录: /data/mysql/logs
    临时目录: /data/mysql/temp
    PID 目录: /data/mysql/run
    所需文件: mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz

2 解决依赖
----------

2.1 安装依赖组件::

    $ yum install libaio perl

2.1 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 3306 mysql

2.3 创建所需目录::

    $ mkdir -pv /program/service/mysql
    $ mkdir -pv /data/service/mysql/{back,conf,data,sbin,logs,vars/{run,tmp}}
    $ chown -R depuser:depuser /program/service/mysql
    $ chown -R svcuser:svcuser /data/service/mysql

3 安装程序
----------

3.1 解压软件包::

    $ cd ~/packages
    $ tar xf mysql-5.7.20-linux-glibc2.12-x86_64.tar.gz -C /program/service/mysql
    $ cd /program/service/mysql
    $ mv mysql-5.7.20-linux-glibc2.12-x86_64 mysql-5.7.20
    $ ln -sv  mysql-5.7.20/ default

3.2 初始化数据库::

    $ cd /program/service/mysql/mysql-5.7.20
    $ bin/mysqld --initialize-insecure --user=svcuser --basedir=/program/service/mysql/mysql-5.7.20 --datadir=/data/service/mysql/data

3.3 创建所需文件::

    $ cd /program/service/mysql/mysql-5.7.20
    $ touch /data/service/mysql/conf/my.cnf
    $ mv /etc/my.cnf /etc/my.cnf.bak-$(date +%s)
    $ ln -sv /data/service/mysql/conf/my.cnf /etc/my.cnf
    $ sed -i 's#/usr/local/mysql#/program/service/mysql/default#g' bin/mysqld_safe               # 修改脚本中程序路径
    $ sed -i 's#/usr/local/mysql#/program/service/mysql/default#g' support-files/mysql.server    # 修改脚本中程序路径
    $ cp support-files/mysql.server /data/service/mysql/sbin

4 修改配置
----------

4.1 编辑配置文件:

.. code-block:: bash

    $ vim /data/service/mysql/conf/my.cnf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    [client]
    port                   = 3306
    socket                 = /data/service/mysql/vars/tmp/mysql.sock

    [mysqld_safe]
    log_error              = /data/service/mysql/logs/mysql.err
    pid_file               = /data/service/mysql/vars/run/mysqld.pid

    [mysqld]
    user                   = svcuser
    port                   = 3306
    basedir                = /program/service/mysql/default
    datadir                = /data/service/mysql/data
    log_error              = /data/service/mysql/logs/mysql.err
    tmpdir                 = /data/service/mysql/vars/tmp
    socket                 = /data/service/mysql/vars/tmp/mysql.sock
    pid_file               = /data/service/mysql/vars/run/mysqld.pid
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
    
    $ cd /program/service/mysql/default
    $ setsid bin/mysqld_safe --defaults-file=/data/service/mysql/conf/my.cnf &>/dev/null

5.2 SysV启动脚本::

    $ service mysql start
    $ cd /data/service/mysql
    $ runuser svcuser -s /bin/bash -c "sbin/mysql.server start"

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
    $ mysql -uroot -pylzone                               # 连接测试

6.2 开机启动::

    $ chkconfig --add mysql
    $ chkconfig mysql on

    $ chmod u+x /etc/rc.d/rc.local
    $ vim /etc/rc.local

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    runuser svcuser -s /bin/bash -c "/data/service/mysql/sbin/mysql.server start"

6.3 添加PATH::
    
    $ echo 'export MYSQL_HOME=/program/service/mysql/default' >/etc/profile.d/mysql.sh
    $ echo 'export PATH=${PATH}:${MYSQL_HOME}/bin' >> /etc/profile.d/mysql.sh
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
