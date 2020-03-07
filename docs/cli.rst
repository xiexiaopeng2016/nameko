命令行界面
==========

Nameko带有一个命令行界面，以使托管和与服务的交互尽可能容易。

.. _running_a_service:

运行一个服务
-----------

.. code-block:: shell

    $ nameko run <module>[:<ServiceClass>]

发现并运行一个服务类。这将在前台启动服务并运行它，直到进程终止。

可以使用一个 ``--config`` 开关覆盖默认设置

.. code-block:: shell

    $ nameko run --config ./foobar.yaml <module>[:<ServiceClass>]

并提供一个简单的YAML配置文件：

.. code-block:: yaml

    # foobar.yaml

    AMQP_URI: 'pyamqp://guest:guest@localhost'
    WEB_SERVER_ADDRESS: '0.0.0.0:8000'
    rpc_exchange: 'nameko-rpc'
    max_workers: 10
    parent_calls_tracked: 10

    LOGGING:
        version: 1
        handlers:
            console:
                class: logging.StreamHandler
        root:
            level: DEBUG
            handlers: [console]


``LOGGING`` 条目将被传递到 :func:`logging.config.dictConfig` ，并应符合该调用的架构。

可以通过内置的 :ref:`config_dependency_provider` 读取配置值。

环境变量替换
-----------

YAML配置文件支持环境变量。您可以使用bash样式语法: ``${ENV_VAR}`` 。可选的，您可以选择提供默认值 ``${ENV_VAR:default_value}`` 。默认值可以递归包含环境变量 ``${ENV_VAR:default_${OTHER_ENV_VAR:value}}`` (注意：此功能需要regex软件包)。


.. code-block:: yaml

    # foobar.yaml
    AMQP_URI: pyamqp://${RABBITMQ_USER:guest}:${RABBITMQ_PASSWORD:password}@${RABBITMQ_HOST:localhost}

运行您的服务并为其设置要使用的环境变量：

.. code-block:: shell

    $ RABBITMQ_USER=user RABBITMQ_PASSWORD=password RABBITMQ_HOST=host nameko run --config ./foobar.yaml <module>[:<ServiceClass>]

如果需要引用YAML文件中的值，则需要显式 ``!env_var`` 解析器：

.. code-block:: yaml

    # foobar.yaml
    AMQP_URI: !env_var "pyamqp://${RABBITMQ_USER:guest}:${RABBITMQ_PASSWORD:password}@${RABBITMQ_HOST:localhost}"

如果您需要将值用作YAML文件中的原始字符串，而无需将其转换为本地python，则需要显式 ``!raw_env_var`` 解析器：

.. code-block:: yaml

    # foobar.yaml
    ENV_THAT_IS_NEEDED_RAW: !raw_env_var "${ENV_THAT_IS_NEEDED_RAW:1234.5660}"

这将变成字符串值 ``1234.5660`` ，而不是浮点数。

您可以提供多个级别的默认值

.. code-block:: yaml

    # foobar.yaml
    AMQP_URI: ${AMQP_URI:pyamqp://${RABBITMQ_USER:guest}:${RABBITMQ_PASSWORD:password}@${RABBITMQ_HOST:localhost}}

这个配置接受AMQP_URI作为一个环境变量，如果提供了RABBITMQ_*嵌套变量将不会被使用。
环境变量值被解释为YAML，因此可以使用丰富的类型：

.. code-block:: yaml

    # foobar.yaml
    ...
    THINGS: ${A_LIST_OF_THINGS}

.. code-block:: shell

    $ A_LIST_OF_THINGS=[A,B,C] nameko run --config ./foobar.yaml <module>[:<ServiceClass>]

环境变量的解析器会将所有方括号配对。

.. code-block::  yaml

    # foobar.yaml
    LANDING_URL_TEMPLATE: ${LANDING_URL_TEMPLATE:https://example.com/{path}}

因此此配置的默认值为 https://example.com/{path}

与正在运行的服务进行交互
-----------------------

.. code-block:: pycon

    $ nameko shell

启动一个交互式python shell，用来于远程nameko服务一起工作。这是一个常规的交互式解释器，在内置名称空间中添加了一个特殊模块 ``n`` ，提供发出RPC调用和调度事件的能力。

对 "target_service" 进行一个RPC调用：

.. code-block:: pycon

    $ nameko shell
    >>> n.rpc.target_service.target_method(...)
    # RPC response

将一个事件调度为"source_service"：

.. code-block:: pycon

    $ nameko shell
    >>> n.dispatch_event("source_service", "event_type", "event_payload")

