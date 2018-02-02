====================
[部署]-编译部署Nginx
====================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: Centos 6.9
    系统内核: Linux version 2.6.32-696.el6.x86_64
    部署方式: compile
    操作用户: root
    运行用户: nginx
    软件版本: nginx-1.12.2
    所需文件: nginx-1.12.2.tar.gz

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - nginx
    
0.3 目录说明::

    程序目录: /program/service/nginx
    备份目录: /data/service/nginx/back
    配置目录: /data/service/nginx/conf
    数据目录: /data/service/nginx/data
    日志目录: /data/service/nginx/logs
    临时目录: /data/service/nginx/vars/tmp
    运行目录: /data/service/nginx/vars/run

0.4 端口说明::

    80: 默认端口，提供HTTP服务所用。

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进度，蓝色为说明提示。

功能点概述::
    
    1. 错误页面重定向
    2. 维护页面配置项


一、解决依赖
------------

1.1 安装依赖组件::

    $ yum install gcc \
                  pcre pcre-devel \
                  zlib zlib-devel \
                  openssl openssl-devel

1.2 创建运行用户::

    $ useradd -M -s /sbin/nologin -u 80 nginx


二、安装程序
------------

2.1 解压编译软件::

    $ cd /tmp
    $ tar xf nginx-1.12.2.tar.gz
    $ cd nginx-1.12.2
    $ ./configure --prefix=/data/service/nginx/data \
                  --conf-path=/data/service/nginx/conf/nginx.conf \
                  --sbin-path=/program/service/nginx/sbin/nginx \
                  --modules-path=/program/service/nginx/lib/modules \
                  --http-log-path=/data/service/nginx/logs/access.log \
                  --error-log-path=/data/service/nginx/logs/error.log \
                  --pid-path=/data/service/nginx/vars/run/nginx.pid \
                  --lock-path=/data/service/nginx/vars/run/nginx.lock \
                  --http-client-body-temp-path=/data/service/nginx/vars/tmp/client_body \
                  --http-proxy-temp-path=/data/service/nginx/vars/tmp/proxy \
                  --http-fastcgi-temp-path=/data/service/nginx/vars/tmp/fastcgi \
                  --http-uwsgi-temp-path=/data/service/nginx/vars/tmp/uwsgi \
                  --http-scgi-temp-path=/data/service/nginx/vars/tmp/scgi \
                  --user=nginx \
                  --group=nginx \
                  --with-http_ssl_module \
                  --with-http_stub_status_module \
                  --with-http_gzip_static_module
    $ make
    $ make install
    $ echo "version: nginx-1.12.2" >> /program/service/nginx/VERSION.md

2.2 创建所需目录::

    $ mkdir -pv /data/service/nginx/{back,conf,data,logs,vars}
    $ mkdir -pv /data/service/nginx/conf/{conf.d,cert}
    $ mkdir -pv /data/service/nginx/vars/{run,tmp}
    $ mkdir -pv /data/service/nginx/vars/tmp/{client,proxy,fastcgi,uwsgi,scgi}

2.3 创建所需文件::

    $ touch /data/service/nginx/conf/conf.d/vhost.default.conf
    $ touch /data/service/nginx/conf/conf.d/vhost.ylone.com.conf
    $ touch /data/service/nginx/conf/conf.d/vhost.upstream.conf

.. note::

    这两个配置文件 ``vhost.ylone.com.conf`` ``upstream.conf``  为演示所用，具体配置请根据实际环境决定。

2.4 修改文件权限::

    $ chown -R root:root /program/service/nginx
    $ chown -R nginx:nginx /data/service/nginx
    $ chown -R root:root /data/service/nginx/conf
    
2.5 修改环境变量::

    $ echo 'export PATH=$PATH:/program/service/nginx/sbin' > /etc/profile.d/nginx.sh
    $ source /etc/profile.d/nginx.sh

2.6 设置开机启动::

    -

.. warning::

    如果后续准备使用 supervisor 启动，则不要执行 ``2.6步骤``。


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: bash

    $ vim /data/service/nginx/conf/nginx.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    daemon on;
    user   nginx;

    error_log  /data/service/nginx/logs/error.log;
    pid        /data/service/nginx/vars/run/nginx.pid;

    worker_processes     2;
    worker_rlimit_nofile 2048;

    events {
        worker_connections  2048;
    }

    http {

        include       mime.types;
        default_type  application/octet-stream;
    
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        log_format json   '{"@timestamp":"$time_iso8601",'
                          '"host":"$server_addr",'
                          '"clientip":"$remote_addr",'
                          '"size":$body_bytes_sent,'
                          '"responsetime":$request_time,'
                          '"upstreamtime":"$upstream_response_time",'
                          '"upstreamhost":"$upstream_addr",'
                          '"http_host":"$host",'
                          '"url":"$uri",'
                          '"domain":"$host",'
                          '"xff":"$http_x_forwarded_for",'
                          '"referer":"$http_referer",'
                          '"agent":"$http_user_agent",'
                          '"status":"$status"}';
    
        access_log  /data/service/nginx/logs/access.log  main;
        
        server_tokens off;             # 隐藏nginx版本号
    
        sendfile        on;
        keepalive_timeout  65;
    
        gzip  on;                      # 启动内容压缩，有效降低网络流量。
        gzip_min_length 1k;            # 过短的内容压缩效果不佳，压缩过程还会浪费系统资源。
        gzip_buffers 4 16k;
        #gzip_http_version 1.0;
        gzip_comp_level 4;             # 可选值1~9,压缩级别越高压缩率越高，但对系统性能要求越高。
        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;    # 压缩的内容类别
        gzip_vary off;                 # 跟Squid等缓存服务有关，on的话会在Header里增加"Vary: Accept-Encoding"。
        gzip_disable "MSIE [1-6]\.";   # IE6对Gzip不怎么友好，对其关闭Gzip。
    
    
        # -----========= 静态文件缓存 ============----- #
        #
        
        open_file_cache max=65535 inactive=20s;    # 最大缓存数量，文件未使用存活期
        open_file_cache_valid 30s;                 # 验证缓存有效期时间间隔
        open_file_cache_min_uses 2;                # 有效期内文件最少使用次数
    

        # -----========= Client默认配置 ============----- #
        #
        
        client_header_buffer_size    128k;         # 设定请求缓冲
        large_client_header_buffers  4 128k;       # 如果 client_header_buffer_size 不够用则使用此。
        client_max_body_size 10m;                  # 客户POST主体安全限额
        client_body_buffer_size 128k;
    

        # -----========= Proxy默认配置 ============----- #
        #
        proxy_redirect off;
        proxy_set_header HOST $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout 90;
        proxy_send_timeout 90;
        proxy_read_timeout 90;
        proxy_buffer_size 4k;
        proxy_buffers 4 32k;
        proxy_busy_buffers_size 64k;
        proxy_temp_file_write_size 64k;
    

        # -----========= 导入扩展配置 ============----- #
        #
        include /data/service/nginx/conf/conf.d/*.conf;

    }

.. code-block:: bash

    $ vim /data/service/nginx/conf/conf.d/vhost.default.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    server {

        listen       80 default_server;
        server_name  _;
    
        location / {
            root   html;
            index  index.html index.htm;
        }
    
        error_page  404              /404.html;
        error_page  500 502 503 504  /50x.html;
    
        location = /50x.html {
            root   html;
        }

    }

.. code-block:: bash

    $ vim /data/service/nginx/conf/conf.d/vhost.ylzone.com.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    server {

        listen 80;
        server_name  www.ylzone.com;

        # -----======================== 维护页面 ========================----- #
        #                                                                      #
        #       说明: 网站维护时开启此配置。                                   #
        #       效果: 客户端访问任何地址调到配置指定页面。                     #
        #       提示: 方式选取一种即可(推荐方式一)，另一种注释掉。             #
        #             方式一，返回用户302重定向指定地址，地址栏变化为指定地址。#
        #             方式二，返回用户指定页面内容，地址栏无变化。             #
        #                                                                      #
        # -------------------------------------------------------------------- #
        #                                                                      #
        #if ($request_uri !~ "^/503.html$") {     # 方式一                     #
        #    rewrite ^(.*)$ /503.html redirect;                                #
        #}                                                                     #
        # -------------------------------------------------------------------- #
        #                                                                      #
        #rewrite ^(.*)$ /503.html break;          # 方式二                     #
        #                                                                      #
        # -----==========================================================----- #

        location / {
            proxy_pass http://www.ylzone.com;
        }

        error_page 503 /503.html;
    
        location = /503.html {
            root   html;
        }
    }

.. code-block:: bash

    $ vim /data/service/nginx/conf/conf.d/upstream.conf

    ↓ ↓ ↓ ↓ ↓ 替换如下内容 ↓ ↓ ↓ ↓ ↓
    upstream www.ylzone.com {

        ip_hash;
        server 192.168.182.145:8080 weight=1;
        server 192.168.182.146:8080 weight=1;

    }


四、启动程序
------------

4.1 启动之前操作:

修改内核配置:

.. code-block:: bash

    # nginx的默认backlog为511，此值受内核somaxconn限制。
    echo 511 > /proc/sys/net/core/somaxconn

4.2 启动应用程序:
    
二进制启动::

    $ /program/service/nginx/sbin/nginx

SysV启动脚本::

    $ service nginx start

supervisor启动配置:

.. code-block:: bash

    [program:nginx]
    command=/program/service/nginx/sbin/nginx -g "daemon off"
    stdout_logfile=/data/service/nginx/logs/supervisor.out
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10
    redirect_stderr=true

.. note::
    
    选择一种启动方式即可，一般使用SysV启动脚本启动即可。

4.3 检测启动状态::

    .. code-block:: bash

    # 测试服务是否启动成功。
    $ curl http://127.0.0.1:80

    # 测试Gzip功能是否开启成功。
    $ curl -I -H "Accept-Encoding: gzip" 127.0.0.1:80/
    HTTP/1.1 200 OK
    Server: nginx
    Date: Mon, 04 Sep 2017 16:48:25 GMT
    Content-Type: text/html
    Last-Modified: Mon, 04 Sep 2017 10:30:04 GMT
    Connection: keep-alive
    ETag: W/"59ad2b2c-264"
    Content-Encoding: gzip

.. note::
    
    测试Gzip功能时，可指定请求资源为 css,js,png 等格式进行测试。


五、附属功能
------------

5.1 安装附属功能:

.. code-block:: bash

    # 为nginx配置安装vim语法着色
    $ cp /tmp/nginx/nginx.vim /usr/share/vim/vim74/syntax/

    # 修改vim相关配置文件
    $ vim /usr/share/vim/vim74/filetype.vim

    ↓ ↓ ↓ ↓ ↓ 追加如下内容 ↓ ↓ ↓ ↓ ↓
    au BufRead,BufNewFile /data/service/nginx/conf/*,/data/service/nginx/conf/conf.d/* if &ft == '' | setfiletype nginx | endif

