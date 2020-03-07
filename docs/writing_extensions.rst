.. _writing_extensions:

编写扩展
==================

结构
---------

扩展应该是 :class:`nameko.extensions.Extension` 的子类。该基类提供了一个扩展的基本结构，特别是下面的方法，可以通过重写来添加功能：

.. sidebar:: 绑定

    一个服务类中的一个扩展仅仅是一个声明。当一个服务是由一个 :ref:`服务容器<containers>` :ref:`主持 <running_services>`，它的扩展是"绑定"到容器的。
	
	对于开发新扩展的开发人员而言，绑定过程是透明的。唯一的考虑应该是， :meth:`~nameko.extensions.Extension.__init__` 是在 :meth:`~nameko.extensions.Extensions.bind` 中以及在服务类声明时调用，因此您应避免此方法的副作用，而应使用 :meth:`~nameko.extensions.Extensions.setup` 代替。
	
.. automethod:: nameko.extensions.Extension.setup

.. automethod:: nameko.extensions.Extension.start

.. automethod:: nameko.extensions.Extension.stop


编写依赖提供者
----------------------------

几乎每个Nameko应用程序都需要定义自己的依赖 - 可能与一个没有 :ref:`社区扩展 <community_extensions>` 的数据库进行交互，或者与一个 :ref:`特定的Web服务 <travis>` 进行通信。

依赖提供程序应该是 :class:`nameko.extensions.DependencyProvider` 的子类，并实现一个 :meth:`~nameko.extensions.DependencyProvider.get_dependency` 方法，该方法返回要注入到服务工作者中的对象。

推荐的模式是为依赖性注入最小的必需接口。这减少了测试范围，并使执行被测服务代码更容易。

依赖提供者也可能会钩(hook)入 *工作者的生命周期* 。为每个工作者在所有依赖提供程序上调用以下三个方法：

.. automethod:: nameko.extensions.DependencyProvider.worker_setup

.. automethod:: nameko.extensions.DependencyProvider.worker_result

.. automethod:: nameko.extensions.DependencyProvider.worker_teardown

并发和线程安全
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

由 :meth:`~nameko.extensions.DependencyProvider.get_dependency` 返回的对象应该是线程安全的，因为它可能被多个并发运行的工作者访问。

*工作者的生命周期* 是在执行服务方法的相同线程调用的。这意味着，例如，您可以定义线程局部变量并从每个方法访问它们。

例子
^^^^^^^

一个简单的 ``DependencyProvider`` 将消息发送到一个SQS队列。

.. literalinclude:: examples/sqs_send.py


编写入口点
-------------------

可以实现新的Entrypoint扩展，如果你希望支持用于初始化服务代码的新传输或机制。

一个入口点的最低要求是：

1. 继承自 :class:`nameko.extensions.Entrypoint`
2. 实现 :meth:`~nameko.extensions.Entrypoint.start()` 方法来启动入口点， 当容器工作时。如果需要一个后台线程，建议使用一个由服务容器管理的线程(请参阅 :ref:`spawning_background_threads`)
3. 在绑定的容器上调用 :meth:`~nameko.containers.ServiceContainer.spawn_worker()`，在适当的时候。


例子
^^^^^^^

一个简单的 ``Entrypoint`` ，它接收来自一个SQS队列的消息。

.. literalinclude:: examples/sqs_receive.py

在一个服务中使用：

.. literalinclude:: examples/sqs_service.py


预期的异常
^^^^^^^^^^^^^^^^^^^

入口点基类构造函数将接受一个类列表，如果这些类是由修饰的服务方法引发的，则应该认为它们是"预期的"。
这可以用来区分 *用户错误* 和更基本的执行错误。例如：

.. literalinclude:: examples/expected_exceptions.py
    :pyobject: Service

预期异常的列表将保存到入口点实例，以便以后可以检查它们，例如，由其他处理异常的扩展，例如 `nameko-sentry <https://github.com/mattbennett/nameko-sentry/blob/b254ba99df5856030dfcb1d13b14c1c8a41108b9/nameko_sentry.py#L159-L164>`_ 。


敏感参数
^^^^^^^^^^^^^^^^^^^

与 *预期的异常* 相同，入口点构造函数允许您将某些参数或部分参数标记为敏感。例如：

.. literalinclude:: examples/sensitive_arguments.py
    :pyobject: Service

可以将其与实用程序函数  :func:`nameko.utils.get_redacted_args` 结合使用，将返回一个入口点的调用args(类似于 :func:`inspect.getcallargs`)，但对敏感元素进行了编辑。

这在记录或保存有关入口点调用信息的扩展中很有用，例如 `nameko-tracer <https://github.com/Overseas-Student-Living/nameko-tracer>`_ 。

对于接受嵌套在其他安全参数中的敏感信息的入口点，可以指定部分编辑。例如：

.. code-block:: python

    # by dictionary key
    @entrypoint(sensitive_arguments="foo.a")
    def method(self, foo):
        pass

    >>> get_redacted_args(method, foo={'a': 1, 'b': 2})
    ... {'foo': {'a': '******', 'b': 2}}

    # list index
    @entrypoint(sensitive_arguments="foo.a[1]")
    def method(self, foo):
        pass

    >>> get_redacted_args(method, foo=[{'a': [1, 2, 3]}])
    ... {'foo': {'a': [1, '******', 3]}}

不支持切片和相对列表索引。

.. _spawning_background_threads:

产生后台线程
---------------------------

需要在线程中执行工作的扩展可以选择使用 :meth:`~nameko.containers.ServiceContainer.spawn_managed_thread()` 将该线程的管理委派给服务容器。

.. literalinclude:: examples/sqs_receive.py
    :pyobject: SqsReceive.start

Delegating thread management to the container is advised because:

* Managed threads will always be terminated when the container is stopped or killed.
* Unhandled exceptions in managed threads are caught by the container and will cause it to terminate with an appropriate message, which can prevent hung processes.
