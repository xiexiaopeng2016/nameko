.. _built_in_dependency_providers:

内置的依赖提供者
=============================

Nameko包括一些常用的 :ref:`依赖项提供程序 <dependency_injection>` 。本节介绍它们，并给出其用法的简要示例。

.. _config_dependency_provider:

配置
------

配置是一个简单的依赖提供者，它在运行时为服务提供对配置值的只读访问权限，请参阅 :ref:`running_a_service` 。

.. literalinclude:: examples/config_dependency_provider.py
