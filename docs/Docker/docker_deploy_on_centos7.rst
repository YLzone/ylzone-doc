==========================
[部署]-CentOS7下部署Docker
==========================


零、背景介绍
------------

0.1 部署说明::
    
    操作系统: 7.3.1611 (Core)
    系统内核: Linux version 3.10.0-514.el7.x86_64
    部署方式: rpm
    操作用户: root
    运行用户: root
    软件版本: 17.06.2-ce
    所需文件: -

0.2 节点说明:

.. list-table::
  :widths: 10 10 30
  :header-rows: 1

  * - Hostname
    - Address
    - Role
  * - VM01
    - 192.168.182.101
    - dockerd, docker
    
0.3 目录说明::

    程序目录: /usr/bin
    启动配置: /usr/lib/systemd/system/docker
    配置文件: /etc/docker, /etc/default/docker
    数据目录: /var/lib/docker
    日志文件: /var/log/messages
    运行文件: /var/run/docker.pid

0.4 端口说明::

    -

0.5 补充说明::

    执行步骤前请查看此步骤下方提示，如遇到红色提示要慎重操作此步骤可能影响正常进度，蓝色为说明提示。

0.5.1 相关资源::

    官方文档: https://docs.docker.com/engine/installation/
    官方仓库: https://download.docker.com/
    检测脚本: https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh


一、解决依赖
------------

1.2 检测系统环境:

.. code-block:: bash

    # 运行环境检测脚本，查看结果获取兼容性。
    $ curl -s https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh | sh

    warning: /proc/config.gz does not exist, searching other paths for kernel config ...
    info: reading kernel config from /boot/config-3.10.0-514.el7.x86_64 ...

    Generally Necessary:
    - cgroup hierarchy: properly mounted [/sys/fs/cgroup]
    - CONFIG_NAMESPACES: enabled
    ......

1.2 安装官方仓库:

.. code-block:: bash

    # 使用命令添加官方repo源
    $ yum install yum-utils
    $ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    
    # 建立yum包索引
    $ yum makecache fast

    # 列出可用版本
    $ yum list docker-ce.x86_64  --showduplicates | sort -r

     Loading mirror speeds from cached hostfile
     Loaded plugins: fastestmirror
     Installed Packages
      * extras: mirror.bit.edu.cn
      * epel: mirror01.idc.hinet.net
     docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable 
     docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable 
     docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable 
     docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable 
     docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable 
     docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 


二、安装程序
------------

2.1 安装指定软件:
    
.. code-block:: bash

    $ yum install docker-ce-17.06.2

2.2 设置开机启动::

    -


三、修改配置
------------

3.1 编辑配置文件:

.. code-block:: bash

    # 修改启动配置，使其支持环境变量。
    $ vim /usr/lib/systemd/system/docker.service

    ↓ ↓ ↓ ↓ ↓ 修改如下内容 ↓ ↓ ↓ ↓ ↓
    EnvironmentFile=/etc/default/docker
    ExecStart=/usr/bin/dockerd  $DOCKER_OPTS


    # 修改启动参数。
    $ vim /etc/default/docker

    ↓ ↓ ↓ ↓ ↓ 修改如下内容 ↓ ↓ ↓ ↓ ↓
    DOCKER_OPTS='--insecure-registry 192.168.182.101:5000'


四、启动程序
------------

4.1 启动之前操作::

    -

4.2 启动应用程序:
    
.. code-block:: bash

    $ systemctl start docker.service
    
4.3 检测启动状态:

.. code-block:: bash

    $ systemctl start docker.service

       ● docker.service - Docker Application Container Engine
      Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
      Active: active (running) since 四 2017-09-07 14:58:14 CST; 18s ago
        Docs: https://docs.docker.com
    Main PID: 7743 (dockerd)
      Memory: 13.8M
      CGroup: /system.slice/docker.service
              ├─7743 /usr/bin/dockerd
              └─7749 docker-containerd -l unix:///var/run/docker/libcontainerd/docker-containerd.sock
     
4.4 启动后续操作:

.. code-block:: bash
    
    # 运行实测容器检测部署
    $ docker run hello-world


五、附属功能
------------

5.1 部署docker镜像操作:

.. code-block:: bash

    # --=== 创建私有仓库  ===--

    # 1. 拉去官方镜像registry。
    $ docker pull registry

    # 2. 启动registry容器，并挂在本地目录到容器。
    $ docker run -d -p 5000:5000 -v /data/docker/volume/registry:/tmp/registry registry

    # 3. 测试连通性。
    $ curl "http://10.0.8.2:5000/v2"
    <a href="/v2/">Moved Permanently</a>.


    # --=== 向私有仓库push镜像 ===--

    # 1. 标记一个镜像
    $ docker tag 1815c82652c0 192.168.182.101:5000/test_push

    # 2. 推送镜像到私有仓库
    $ docker push 192.168.182.101:5000/test_push

    # 3. 查看仓库情况
    $ curl 10.0.8.2:5000/v2/_catalog
    {"repositories":["test1"]}

    # 4. 拉取一个镜像到本地
    $ docker pull 10.0.8.2:5000/test_push
    Using default tag: latest
    latest: Pulling from test1
    Digest: sha256:9fa82f24cbb11b6b80d5c88e0e10c3306707d97ff862a3018f22f9b49cef303a
    Status: Downloaded newer image for 10.0.8.2:5000/test_push:lates


