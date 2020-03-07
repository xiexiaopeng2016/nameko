关键概念
========

本节介绍Nameko的核心概念。

服务剖析
-------

一个Nameko服务只是一个Python类。该类将应用程序逻辑封装在其方法中，并将所有 :ref:`依赖 <dependencies>` 声明为属性。

方法通过 :ref:`入口点 <entrypoints>` 装饰器暴露给外部世界。

.. literalinclude:: examples/anatomy.py

.. _entrypoints:

入口点
^^^^^^

入口点是进入它们装饰的服务方法的网关。它们通常监视一个外部实体，例如消息队列。在相关事件上，入口点可能会“开火”，并且修饰的方法将由一个服务 :ref:`工作者<workers>` 执行。

.. _dependencies:

依赖
^^^^

大多数服务都依赖于自己以外的东西。Nameko鼓励将这些东西实现为依赖。

一个依赖是隐藏不属于核心服务逻辑的代码的一个机会。服务的依赖的接口应尽可能简单。

出于 :ref:`多种原因 <benefits_of_dependency_injection>`，在服务中声明依赖是一个好主意，您应该把它们看作是服务代码和其他所有东西之间的网关。这包括其他服务、外部api，甚至数据库。

.. _workers:

工作者
^^^^^^

入口点触发时创建工作者。一个工作者只是服务类的一个实例，但是依赖声明已替换为那些依赖的实例(请参阅 :ref:`依赖注入 <dependency_injection>`)。

请注意，一个工作者只为执行一个方法而存在 - 服务从一次调用到下一次调用是无状态的，这鼓励了依赖的使用。

一个服务可以同时运行多个工作者，最多可以达到用户定义的限制。有关详细信息，请参见 :ref:`并发 <concurrency> 。

.. _dependency_injection:

依赖注入
--------

向一个服务类添加一个依赖是声明性的。也就是说，类上的属性是一个声明，而不是工作者可以实际使用的接口。。

类属性是一个 :class:`~nameko.extensions.DependencyProvider`。它负责提供一个注入到服务工作者中的对象。

依赖提供程序实现一个 :meth:`~nameko.extensions.DependencyProvider.get_dependency` 方法，其结果被注入到一个新创建的工作者中。

一个工作者的生命周期为：

    #. 入口点触发
    #. 从服务类实例化工作者
    #. 依赖注入到工作者
    #. 方法执行
    #. 销毁工作者

用伪代码看起来像::

    worker = Service()
    worker.other_rpc = worker.other_rpc.get_dependency()
    worker.method()
    del worker

依赖提供者在服务期间有效，而注入的依赖对于每个工作者而言都是唯一的。

.. _concurrency:

并发
----

Nameko建立在 `eventlet <http://eventlet.net/>`_ 库之上，该库通过"greenthreads"提供并发性。并发模型是隐式生成的协同例程。

隐式良率依赖于 `猴子补丁 <http://eventlet.net/doc/patching.html#monkeypatching-the-standard-library>`_ 标准库，在线程等待I/O时触发一个yield。如果您在命令行上使用 ``nameko run`` 托管服务，Nameko将为您应用猴子补丁。

每个工作者都在其自己的greenthread中执行。可以根据每个工作者在I/O上等待所花费的时间来调整并发工作者的最大数量。

工作者是无状态的，所以线程本质上是安全的，但是依赖关系应确保它们对于每个工作者都是唯一的，否则，可以安全地由多个工作者同时访问。

请注意，许多正在使用套接字的C-扩展名通常被认为是线程安全的，可能不适用于greenthreads。其中有 `librabbitmq <https://pypi.python.org/pypi/librabbitmq>`_, `MySQLdb <http://mysql-python.sourceforge.net/MySQLdb.html>`_ 和其它。

.. _extensions:

扩展
----

所有入口点和依赖提供程序都实现为"扩展"。我们之所以这样称呼它们，是因为它们在服务代码之外，但不是所有服务都需要它们(例如，纯AMQP公开的服务将不使用HTTP入口点)。

Nameko具有许多 :ref:`内置扩展<built_in_extensions>` ，其中一些 :ref:`由社区提供<community_extensions>`，您可以 :ref:`编写自己的扩展<writing_extensions>` 。

.. _running_services:

运行服务
-------

运行一个服务所需的全部就是服务类和任何相关配置。运行一个或多个服务的最简单方法是使用Nameko CLI::

    $ nameko run module:[ServiceClass]

此命令将在给定 ``module`` 中发现Nameko服务并开始运行它们。您可以选择将其限制为特定的 ``ServiceClass`` 。

.. _containers:


服务容器
^^^^^^^^

每个服务类都委托给一个 :class:`~nameko.containers.ServiceContainer` 。容器封装了运行服务所需的所有功能，并且还封装了服务类上的任何 :ref:`扩展<extensions>` 。

使用 ``ServiceContainer`` 来运行一个单一的服务：

.. literalinclude:: examples/service_container.py

.. _runner:

服务运行器
^^^^^^^^^^

:class:`~nameko.runners.ServiceRunner` 是一个围绕多个容器的薄包装，它公开了同时启动和停止所有包装容器的方法。这是 ``nameko run`` 内部使用的东西，也可以通过编程方式构造：

.. literalinclude:: examples/service_runner.py

如果你创建自己的运行程序而不是使用 `nameko run` ，则还必须应用eventlet `猴子补丁 <http://eventlet.net/doc/patching.html#monkeypatching-the-standard-library>`_ 。

有关示例，请参见 `nameko.cli.run <https://github.com/nameko/nameko/blob/cc13802d8afb059419384e2e2016bae7fe1415ce/nameko/cli/run.py#L3-L4>`_ 模块。
