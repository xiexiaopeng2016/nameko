测试服务
================

哲学
----------

Nameko的约定旨在使测试尽可能简单。服务可能很小且用途单一，并且依赖注入使替换和隔离功能块变得很容易。

下面的示例使用 `pytest <http://pytest.org/latest/>`_ ，这是Nameko自己的测试套件使用的东西，但是辅助工具与测试框架无关。


单元测试
------------

Nameko中的单元测试通常意味着对单个服务进行隔离测试 -- 即没有任何或大部分依赖。

:func:`~nameko.testing.services.worker_factory` 实用程序将从一个给定的服务类创建一个工作者，并将其依赖项替换为 :class:`mock.MagicMock` 对象。然后可以通过添加 :attr:`~mock.Mock.side_effect` 和 :attr:`~mock.Mock.return_value` 来模仿依赖功能：

.. literalinclude:: examples/testing/unit_test.py

在某些情况下，提供另一种依赖关系比使用一个mock更有帮助。这可能是一个功能完整的替换(例如，一个测试数据库会话)或一个提供部分功能的轻量级垫片(shim)。

.. literalinclude:: examples/testing/alternative_dependency_unit_test.py

集成测试
-------------------

Nameko中的集成测试意味着测试许多服务之间的接口。推荐的方法是以正常的方式运行所有被测试的服务，并通过使用一个助手"触发"入口点来触发行为：

.. literalinclude:: examples/testing/integration_test.py

需要注意这里的``ServiceX``和``ServiceY``之间的接口，是像在正常操作下运行。

超出特定测试范围的接口可以通过以下测试帮助程序之一停用:

restrict_entrypoints
^^^^^^^^^^^^^^^^^^^^

.. autofunction:: nameko.testing.services.restrict_entrypoints
   :noindex:

replace_dependencies
^^^^^^^^^^^^^^^^^^^^

.. autofunction:: nameko.testing.services.replace_dependencies
   :noindex:

完整的示例
^^^^^^^^^^^^^^^^

下面的集成测试示例使用了两个范围限制助手：

.. literalinclude:: examples/testing/large_integration_test.py

其它助手
-------------

entrypoint_hook
^^^^^^^^^^^^^^^

入口点挂钩允许手动调用一个服务入口点。如果对导致入口点触发的外部事件进行伪造非常困难或代价昂贵，这在集成测试期间非常有用。

您可以为调用提供 `context_data` 来模拟特定的调用上下文，例如语言、用户代理或验证令牌。

.. literalinclude:: examples/testing/entrypoint_hook_test.py

entrypoint_waiter
^^^^^^^^^^^^^^^^^

入口点等待程序是一个上下文管理器，它在一个命名的入口点已触发并完成之前不会退出。这在测试异步服务之间的集成点时非常有用，例如接收事件：

.. literalinclude:: examples/testing/entrypoint_waiter_test.py

请注意，上下文管理器不仅等待入口点方法完成，而且还等待任何依赖项拆除。例如，基于依赖的日志记录器，例如(TODO:链接到绑定的日志记录器)也已经完成。

使用pytest
------------

Nameko的测试套件使用pytest，如果您选择使用pytest，则可以为自己的测试提供一些有用的配置和装置。

它们包含在 :mod:`nameko.testing.pytest` 中。此模块已使用使用setuptools注册为一个 `pytest插件 <https://docs.pytest.org/en/latest/plugins.html>`_ ，Pytest会自动选择并使用它。
