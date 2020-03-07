.. _built_in_extensions:

内置扩展
========

Nameko包含许多内置 :ref:`扩展<extensions>` 。本节介绍它们，并给出其用法的简要示例。

RPC
---

Nameko包含一个通过AMQP的RPC实现。它包括 ``@rpc`` 入口点，一个用于与其他服务对话的服务的代理，和一个独立的代理，非nameko客户端可以使用它来对一个集群进行RPC调用：

.. literalinclude:: examples/rpc.py

.. literalinclude:: examples/standalone_rpc.py

普通RPC调用会阻塞，直到远程方法完成，但是代理也具有针对后台的异步调用模式或并行化RPC调用：

.. literalinclude:: examples/async_rpc.py

在一个具有多个目标服务实例的群集中，RPC请求实例之间的循环。该请求将仅由目标服务的一个实例处理。

AMQP消息只有在请求被成功处理之后才会被ack。如果服务未能确认消息并且AMQP连接已关闭(例如，如果服务进程被终止)，则代理将撤消，然后将消息分配给可用的服务实例。

请求和响应有效负载被序列化为JSON，以便通过网络(wire)进行传输。

事件(Pub-Sub)
-------------

Nameko事件是一个异步消息传递系统，实现了发布-订阅模式。服务调度事件，它可能被零个或多个其他事件接收：

.. literalinclude:: examples/events.py

:class:`~nameko.events.EventHandler 入口点有三个``handler_type``，它表示确定的事件消息是如何在一个集群中收到的：

    * ``SERVICE_POOL`` -- 事件处理程序按服务名称进行池化，每个池中的一个实例接收事件，类似于RPC入口点的群集行为。这是默认的处理程序类型。
    * ``BROADCAST`` -- 每个侦听服务实例将接收该事件。
    * ``SINGLETON`` -- 仅一个监听服务实例将接收该事件。

一个使用 ``BROADCAST`` 模式的示例：

.. literalinclude:: examples/event_broadcast.py

事件被序列化为JSON以通过网络(wire)传输。

HTTP
---------------

HTTP入口点建立在 `werkzeug <http://werkzeug.pocoo.org/>`_ 之上，并支持所有标准HTTP方法(GET/POST/DELETE/PUT等)

HTTP入口点可以将单个URL的多个HTTP方法指定为逗号分隔的列表。请参见下面的示例。

服务方法必须返回以下之一：

- 一个字符串，它成为响应主体
- 一个2元组 ``(status code, response body)``
- 一个3元组 ``(status_code, headers dict, response body)``
- 一个 :class:`werkzeug.wrappers.Response` 实例

.. literalinclude:: examples/http.py

.. code-block:: shell

    $ nameko run http
    starting services: http_service

.. code-block:: shell

    $ curl -i localhost:8000/get/42
    HTTP/1.1 200 OK
    Content-Type: text/plain; charset=utf-8
    Content-Length: 13
    Date: Fri, 13 Feb 2015 14:51:18 GMT

    {'value': 42}

.. code-block:: shell

    $ curl -i -d "post body" localhost:8000/post
    HTTP/1.1 200 OK
    Content-Type: text/plain; charset=utf-8
    Content-Length: 19
    Date: Fri, 13 Feb 2015 14:55:01 GMT

    received: post body

一个更高级的示例：

.. literalinclude:: examples/advanced_http.py

.. code-block:: shell

    $ nameko run advanced_http
    starting services: advanced_http_service

.. code-block:: shell

    $ curl -i localhost:8000/privileged
    HTTP/1.1 403 FORBIDDEN
    Content-Type: text/plain; charset=utf-8
    Content-Length: 9
    Date: Fri, 13 Feb 2015 14:58:02 GMT

.. code-block:: shell

    curl -i localhost:8000/headers
    HTTP/1.1 201 CREATED
    Location: https://www.example.com/widget/1
    Content-Type: text/plain; charset=utf-8
    Content-Length: 0
    Date: Fri, 13 Feb 2015 14:58:48 GMT


您可以通过重写 :meth:`~nameko.web.HttpRequestHandler.response_from_exception` 来控制服务返回的错误的格​​式：

.. literalinclude:: examples/http_exceptions.py

.. code-block:: shell

    $ nameko run http_exceptions
    starting services: http_service

.. code-block:: shell

    $ curl -i http://localhost:8000/custom_exception
    HTTP/1.1 400 BAD REQUEST
    Content-Type: application/json
    Content-Length: 72
    Date: Thu, 06 Aug 2015 09:53:56 GMT

    {"message": "Argument `foo` is required.", "error": "INVALID_ARGUMENTS"}

您可以使用 `WEB_SERVER_ADDRESS` 配置来更改HTTP端口和IP ：

.. code-block:: yaml

    # foobar.yaml

    AMQP_URI: 'pyamqp://guest:guest@localhost'
    WEB_SERVER_ADDRESS: '0.0.0.0:8000'

计时器
------

:class:`~nameko.timers.Timer` 是一个简单的入口点，每可配置的秒数触发一次。计时器不是"群集感知"的，并且会在所有服务实例上触发。

.. literalinclude:: examples/timer.py
