.. _community_extensions:

社区
=========

有许多nameko扩展和补充库不是核心项目的一部分，但是你在开发自己的nameko服务时可能会有用：

扩展
----------

    * `nameko-sqlalchemy <https://github.com/onefinestay/nameko-sqlalchemy>`_
		
		一个 ``DependencyProvider`` 用于使用SQLAlchemy写入数据库。需要一个纯Python或其他兼容Eventlet的数据库驱动程序。
		
		当通过一个REST API公开查询对象时，请考虑将其与 `SQLAlchemy-filters <https://github.com/Overseas-Student-Living/sqlalchemy-filters>`_ 组合以添加查询对象的过滤，排序和分页。
		
    * `nameko-sentry <https://github.com/mattbennett/nameko-sentry>`_
		
		捕获入口点异常并将反馈信息发送到一个 `Sentry <https://getsentry.com/>`_ 服务器。

    * `nameko-amqp-retry <https://github.com/nameko/nameko-amqp-retry>`_
		
		允许AMQP入口点稍后重试的Nameko扩展。

    * `nameko-bayeux-client <https://github.com/Overseas-Student-Living/nameko-bayeux-client>`_
		
		具有一个实现Bayeux协议的Cometd客户端的Nameko扩展

    * `nameko-slack <https://github.com/iky/nameko-slack>`_
		
		用于与Slack API交互的Nameko扩展。使用Python的Slack开发者工具箱。

    * `nameko-eventlog-dispatcher <https://github.com/sohonetlabs/nameko-eventlog-dispatcher>`_
		
		Nameko依赖提供程序，它使用事件(Pub-Sub)调度日志数据。

    * `nameko-redis-py <https://github.com/fraglab/nameko-redis-py>`_
		
		Redis对Nameko的依赖和实用程序。

    * `nameko-redis <https://github.com/etataurov/nameko-redis/>`_
		
		Redis对Nameko服务的依赖

    * `nameko-statsd <https://github.com/sohonetlabs/nameko-statsd>`_
		
		一个StatsD对Nameko的依赖，使服务能够发送统计信息。

    * `nameko-twilio <https://github.com/invictuscapital/nameko-twilio>`_

        Nameko的Twilio依赖，因此您可以在服务中发送短信，拨打电话和接听电话。

    * `nameko-sendgrid <https://github.com/invictuscapital/nameko-sendgrid>`_
		
		Nameko的SendGrid依赖性，用于发送交易和市场营销电子邮件。
        
    * `nameko-cachetools <https://github.com/santiycr/nameko-cachetools>`_
		
		用于缓存nameko服务之间的RPC交互的工具。


补充库
-----------------------

    * `django-nameko <https://github.com/and3rson/django-nameko>`_

        用于Nameko微服务框架的Django包装器。

    * `flask_nameko <https://github.com/clef/flask-nameko>`_

        在Flask中使用nameko服务的包装器。

    * `nameko-proxy <https://github.com/fraglab/nameko-proxy>`_
		
		独立的异步代理与Nameko微服务进行通信。

在PyPi中搜索更多 `nameko软件包 <https://pypi.python.org/pypi?%3Aaction=search&term=nameko&submit=search>`_ ，如果您希望自己的nameko扩展或库出现在此页面上，请 :ref:`联系 <getting_in_touch>` 。
