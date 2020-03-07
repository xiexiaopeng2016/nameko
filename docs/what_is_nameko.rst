什么是Nameko?
============

Nameko是用于在Python中构建微服务的框架。

它内置了对以下内容的支持：

    * 通过AMQP进行RPC
    * 通过AMQP的异步事件(pub-sub)
    * 简单的HTTP GET和POST
    * Websocket RPC和订阅(实验性)

开箱即用，您可以构建一个服务，该服务可以响应RPC消息，调度某些操作的事件以及侦听其他服务的事件。它还可以为无法说(speak)AMQP的客户端提供HTTP接口，为Javascript客户端提供websocket接口。

Nameko也可以扩展。您可以定义自己的传输机制和服务依赖，以根据需要进行混合和匹配。

Nameko强烈建议使用 :ref:`依赖注入<benefits_of_dependency_injection>` 模式，这种模式使构建和测试服务简洁明了。

Nameko的名字来自于成群生长的日本蘑菇。

我什么时候应该使用Nameko?
-----------------------

Nameko旨在帮助您创建，运行和测试微服务。如果满足以下条件，则应使用Nameko：

    * 您想要将后端编写为微服务，或者
    * 您想要将微服务添加到现有系统，并且
    * 您想用Python做到这一点。

Nameko从单个服务的单个实例扩展到具有许多不同服务的多个实例的集群。

该库还附带了用于客户端的工具，如果您想编写Python代码与现有的Nameko群集进行通信，则可以使用该工具。


我什么时候不应该使用Nameko?
-------------------------

Nameko不是一个web框架。它具有内置的HTTP支持，但仅限于微服务领域中有用的功能。如果要构建供人类使用的Web应用程序，则应使用 `flask <http://flask.pocoo.org>`_ 。
