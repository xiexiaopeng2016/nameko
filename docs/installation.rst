.. _installation:

安装
====

使用Pip安装
-----------

您可以从 `PyPI <https://pypi.python.org/pypi/nameko>`_ 使用pip安装nameko及其依赖：

    pip install nameko


源代码
------

Nameko在 `GitHub <https://github.com/nameko/nameko>`_ 上积极开发。通过克隆公共存储库获取代码::

    git clone git@github.com:nameko/nameko.git

您可以使用setuptools从源代码进行安装::

    python setup.py install


RabbitMQ
--------

Nameko的一些内置功能依赖RabbitMQ。在大多数平台上安装RabbitMQ都很简单，并且它们具有 `出色的文档 <https://www.rabbitmq.com/download.html>`_ 。

在Mac上，您可以使用Homebrew安装::

    brew install rabbitmq

在基于debian的操作系统上::

    apt-get install rabbitmq-server

对于其他平台，请查阅 `RabbitMQ安装指南 <https://www.rabbitmq.com/download.html>`_ 。

RabbitMQ代理将在安装后立即准备就绪 -- 无需任何配置。本文档中的示例假定您在本地主机的默认端口上运行了代理，并且启用了 `rabbitmq_management <http://www.rabbitmq.com/management.html>`_ 插件。
