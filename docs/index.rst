:orphan:

Nameko
======

*[nah-meh-koh]*

.. pull-quote ::
	
	一个用于Python的微服务框架，允许服务开发人员专注于应用程序逻辑并鼓励可测试性。

一个nameko服务只是一个类:

.. code-block:: python

    # helloworld.py

    from nameko.rpc import rpc

    class GreetingService:
        name = "greeting_service"

        @rpc
        def hello(self, name):
            return "Hello, {}!".format(name)

.. note::
	上面的例子需要 `RabbitMQ <https://www.rabbitmq.com>`_ 因为它使用了内置的AMQP RPC特性。 `RabbitMQ安装指南 <https://www.rabbitmq.com/download.html>`_ 提供几个安装选项，但是您可以使用 `Docker <https://docs.docker.com/install/>`_ 命令快速安装和运行它。

    使用docker安装和运行RabbitMQ:

    .. code-block:: shell

       $ docker run -d -p 5672:5672 rabbitmq:3

    | *您可能需要使用sudo来实现这一点*

你可以在一个shell运行它:

.. code-block:: shell

    $ nameko run helloworld
    starting services: greeting_service
    ...

和另一个shell一起玩：

.. code-block:: pycon

    $ nameko shell
    >>> n.rpc.greeting_service.hello(name="ナメコ")
    'Hello, ナメコ!'



用户指南
--------

本节涵盖了创建和运行自己的Nameko服务所需了解的大部分内容。

.. toctree::
   :maxdepth: 2

   what_is_nameko
   key_concepts
   installation
   cli
   built_in_extensions
   built_in_dependency_providers
   community_extensions
   testing
   writing_extensions



更多信息
--------

.. toctree::
   :maxdepth: 2

   about_microservices
   dependency_injection_benefits
   similar_projects
   getting_in_touch
