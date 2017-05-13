======
Sphinx
======

1 介绍
======
Sphinx 是一种文档工具，它可以令人轻松的撰写出清晰且优美的文档, 由 Georg Brandl 在BSD 许可证下开发. 新版的Python文档 就是由Sphinx生成的， 并且它已成为Python项目首选的文档工具,同时它对 C/C++ 项目也有很好的支持; 并计划对其它开发语言添加特殊支持. 本站当然也是使用 Sphinx 生成的，它采用reStructuredText!

2 安装
======
2.1 使用pip安装::

    $ pip install Sphinx

2.2 安装主题::

    $ pip install sphinx_rtd_theme

3 使用
======
3.1 初始化目录::

    $ mkdir PROJECT_NAME && cd PROJECT_NAME
    $ sphinx-quickstart

3.2 生成的目录结构::

    make.bat        # Windowsx下编译脚本  
    Makefile        # Linux下Makefile文件  
    build           # make编译后产生的网页目录在build/html目录下  
    source          # 文档源码目录  
    conf.py         # 配置文件  
    index.rst       # 文档源文件入口
    static          # 编译过程产生的一些图片之类的
    templates       # 模板 

3.3 生成html静态文档::

    $ cd PROJECT_NAME
    $ make clean    # 清除之前编译make过的内容
    $ make html
