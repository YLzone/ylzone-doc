===============
[安装]-单机编译
===============

1 编译安装
----------------

1.0 部署说明::
    
    操作系统: Centos 6.5
    系统内核: Linux version 2.6.32-431.el6.x86_64
    所需文件: redis-3.2.8.tar.gz
    操作用户: root
    运行用户: redis
    软件版本: redis-3.2.8
    部署位置: /opt/redis-3.2.8
    配置目录: /data/redis/conf
    数据目录: /data/redis/data/redis_6379
    日志目录: /data/redis/logs
    特殊说明: 守护进程、开启dump、关闭aof

    # 维护相关
    检测存活: # redis-cli PING >> PONG


1.1 安装编译依赖环境::

    $ yum -y install gcc gcc-c++

1.2 源码编译程序::

    $ cd /tmp
    $ wget http://download.redis.io/releases/redis-3.2.8.tar.gz
    $ tar xzf redis-3.2.8.tar.gz
    $ cd redis-3.2.8
    $ make

1.3 规范运行用户::

    $ useradd -M -s /sbin/nologin -u 6379 redis  # 建立运行用户

1.4 规范目录创建::
    
    $ mkdir -pv /opt/redis-3.2.8/bin      # 创建可执行文件目录
    $ mkdir -pv /data/redis/conf          # 创建配置文件目录
    $ mkdir -pv /data/redis/data          # 创建数据目录
    $ mkdir -pv /data/redis/logs          # 创建日志目录
    $ mkdir -pv /data/redis/run           # 创建PID目录
    $ ln -sv /opt/redis-3.2.8 /opt/redis               # 创建程序目录软连接
 
1.5 复制文件到指定目录::

    $ cd /tmp/redis-3.2.8/src
    $ cp redis-benchmark redis-trib.rb  /opt/redis-3.2.8/bin
    $ cp redis-check-rdb redis-check-aof /opt/redis-3.2.8/bin
    $ cp redis-server redis-cli redis-sentinel /opt/redis-3.2.8/bin
    $ cp ../redis.conf /data/redis/conf/redis_6379.conf

1.6 修改配置文件redis_6379.conf:

.. code-block:: bash

    ################################## INCLUDES ###################################
    # include /path/to/local.conf
    # include /path/to/other.conf

    ################################## NETWORK #####################################
    bind 0.0.0.0
    protected-mode yes
    port 6379
    tcp-backlog 511
    timeout 0
    tcp-keepalive 300

    ################################# GENERAL #####################################
    daemonize yes
    supervised no
    pidfile "/data/redis/run/redis_6379.pid"
    loglevel notice
    logfile "/data/redis/logs/redis_6379.log"
    databases 16

    ################################ SNAPSHOTTING  ################################
    save 900 1
    save 300 10
    save 60 10000
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename "dump.rdb"
    dir "/data/redis/data/redis_6379"

    ################################# REPLICATION #################################
    slave-serve-stale-data yes
    slave-read-only yes
    repl-diskless-sync no
    repl-diskless-sync-delay 5
    repl-disable-tcp-nodelay no
    slave-priority 100

    ################################## SECURITY ###################################


    ################################### LIMITS ####################################
    # maxclients 10000
    # maxmemory <bytes>
    # maxmemory-samples 5

    ############################## APPEND ONLY MODE ###############################
    appendonly no
    appendfilename "appendonly.aof"
    appendfsync everysec
    no-appendfsync-on-rewrite no
    auto-aof-rewrite-percentage 100
    auto-aof-rewrite-min-size 64mb
    aof-load-truncated yes

    ################################ LUA SCRIPTING  ###############################
    lua-time-limit 5000

    ################################ REDIS CLUSTER  ###############################
    # cluster-enabled yes
    # cluster-config-file nodes-6379.conf
    # cluster-node-timeout 15000
    # cluster-slave-validity-factor 10
    # cluster-migration-barrier 1
    # cluster-require-full-coverage yes

    ################################## SLOW LOG ###################################
    slowlog-log-slower-than 10000
    slowlog-max-len 128

    ################################ LATENCY MONITOR ##############################
    latency-monitor-threshold 0

    ############################# EVENT NOTIFICATION ##############################
    notify-keyspace-events ""

    ############################### ADVANCED CONFIG ###############################
    hash-max-ziplist-entries 512
    hash-max-ziplist-value 64
    list-max-ziplist-size -2
    list-compress-depth 0
    set-max-intset-entries 512
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
    hll-sparse-max-bytes 3000
    activerehashing yes
    client-output-buffer-limit normal 0 0 0
    client-output-buffer-limit slave 256mb 64mb 60
    client-output-buffer-limit pubsub 32mb 8mb 60
    hz 10
    aof-rewrite-incremental-fsync yes

.. note::

    个性化配置相关参数:
    ``port``
    ``pidfile``
    ``logfile``
    ``dir``

1.7 规范目录权限::

    $ chown -R root:root /data/redis/conf         # 给配置目录赋权
    $ chown -R redis:redis /data/redis/data       # 给数据目录赋权
    $ chown -R redis:redis /data/redis/logs       # 给日志目录赋权
    $ chown -R root:root /opt/redis-3.2.8         # 给程序目录赋权
    $ chmod +x /opt/redis-3.2.8/bin/*

.. note::

    可以使用如下命令启动:
    ``$ sudo -u redis /opt/redis/bin/redis-server /data/redis/conf/redis_6379.conf``

1.8 创建启动脚本/etc/init.d/redis_6379:

.. code-block:: bash

    #!/bin/sh
    #
    # Simple Redis init.d script conceived to work on Linux systems
    # as it does use of the /proc filesystem.

    ###############
    # SysV Init Information
    # chkconfig: - 58 74
    # description: redis_6379 is the redis daemon.
    ### BEGIN INIT INFO
    # Provides: redis_6379
    # Required-Start: $network $local_fs $remote_fs
    # Required-Stop: $network $local_fs $remote_fs
    # Default-Start: 2 3 4 5
    # Default-Stop: 0 1 6
    # Should-Start: $syslog $named
    # Should-Stop: $syslog $named
    # Short-Description: start and stop redis_6379
    # Description: Redis daemon
    ### END INIT INFO

    USER="redis"
    BASEDIR="/opt/redis"
    DATADIR="/data/redis"
    REDISPORT="6379"
    EXEC="${BASEDIR}/bin/redis-server"
    CLIEXEC="${BASEDIR}/bin/redis-cli"
    PIDFILE="${DATADIR}/run/redis_${REDISPORT}.pid"
    CONF="${DATADIR}/conf/redis_${REDISPORT}.conf"

    case "$1" in
        start)
            if [ -f $PIDFILE ]
            then
                echo "$PIDFILE exists, process is already running or crashed"
            else
                echo "Starting Redis server..."
                su $USER -c "$EXEC $CONF"
            fi
            ;;
        stop)                                                               
            if [ ! -f $PIDFILE ]                                            
            then                                                            
                echo "$PIDFILE does not exist, process is not running"      
            else                                                            
                PID=$(cat $PIDFILE)
                echo "Stopping ..."                                         
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do                                                          
                    echo "Waiting for Redis to shutdown ..."                
                    sleep 1                                                 
                done
                echo "Redis stopped"
            fi
            ;;
        status)
            PID=$(cat $PIDFILE)
            if [ ! -x /proc/${PID} ]
            then
                echo 'Redis is not running'
            else
                echo "Redis is running ($PID)"
            fi
            ;;
        restart)
            $0 stop
            $0 start
            ;;
        *)
            echo "Please use start, stop, restart or status as first argument"
            ;;
    esac

1.9 使用脚本启动::

    $ chmod +x /etc/init.d/redis_6379
    $ service redis_6379 start
    $ chkconfig --add redis_6379
    $ chkconfig redis_6379 on

1.10 添加环境变量::

    编辑配置文件:
    $ vim /etc/profile.d/redis.sh

    添加如下内容:
    PATH=$PATH:/opt/redis/bin 
    export PATH

    载入环境变量:
    $ source /etc/profile.d/redis.sh
