.. _startup_chapter:

Startup
=======
启动
====


When you cause a :app:`Pyramid` application to start up in a console window,
you'll see something much like this show up on the console:

当你在windows控制台下启动`Pryarmid`应用，你会见到如下提示

.. code-block:: text

  $ pserve myproject/MyProject.ini
  Starting server in PID 16601.
  serving on 0.0.0.0:6543 view at http://127.0.0.1:6543

当你在控制台敲入命令 ``pserve myproject/MyProject.ini`` 后，系统会最终会显示
``serving on 0.0.0.0:6543 ...``,本章将解析在这个过程中的各个细节

This chapter explains what happens between the time you press the "Return"
key on your keyboard after typing ``pserve myproject/MyProject.ini``
and the time the line ``serving on 0.0.0.0:6543 ...`` is output to your
console.

.. index::
   single: startup process

The Startup Process
-------------------
启动过程
--------

最容易和文档化的启动 `Pyramid` 方式是用 ``pserve`` 命令，而不是自己
写 `PasteDeploy` ``.ini`` 文件。这些 ``.ini`` 文件用于系统的相关设置和启动
监听端口( ``pserve`` 命令会自动生成相关的 ``.ini``  )。 为了更好的讨论，我
们假设你是用 ``pserve`` 来启动 `Pyramid` 系统应用。

 
The easiest and best-documented way to start and serve a :app:`Pyramid`
application is to use the ``pserve`` command against a :term:`PasteDeploy`
``.ini`` file.  This uses the ``.ini`` file to infer settings and starts a
server listening on a port.  For the purposes of this discussion, we'll
assume that you are using this command to run your :app:`Pyramid`
application.

下面按步骤说明启动过程

Here's a high-level time-ordered overview of what happens when you press
``return`` after running ``pserve development.ini``.


#.  ``pserve`` 启动时候指定了参数   ``development.ini`` 文件. Pyramid 会
      读取文件里面的控制信息来启动自己。  


#. The ``pserve`` command is invoked under your shell with the argument
   ``development.ini``.  As a result, Pyramid recognizes that it is meant to
   begin to run and serve an application using the information contained
   within the ``development.ini`` file.

#. Pyramid查找配置文件 ``.ini``  中名为 ``[app:main]``, ``[pipeline:main]`` 
     或  ``[composite:main]`` 部分，使用来它配置系统 `WSGI` 功能的。
     在简单的服务应用中，系统使用  ``[app:main]`` 配置，行 ``use=`` 定义了
     系统的入口点 `entry point` 或   点模块名 `dotted Python name`。如果
     不是简单服务应用，而是用 WSGI 的管道方式，系统将使用 ``[pipeline:main]``
     配置，管道服务按命名顺序最终调用 `Pyramid` 服务，pipeline主要用于filter
     例如  pipeline = filter1 egg:FilterEgg#filter2 filter3 app ，app就是本应用
     服务（参考 http://blog.csdn.net/sonicatnoc/article/details/6539716 ）。 最后
      ``[composite:main]``  配置了一些特殊Pyramid 组件。 在大多数情况下,  
      Pyramid 的  scaffold 自动生成的配置只有一个 ``[app:main]`` 段，并用它
      来配置构建自身服务。


#. The framework finds a section named either ``[app:main]``,
   ``[pipeline:main]``, or ``[composite:main]`` in the ``.ini`` file.  This
   section represents the configuration of a :term:`WSGI` application that
   will be served.  If you're using a simple application (e.g.
   ``[app:main]``), the application :term:`entry point` or :term:`dotted
   Python name` will be named on the ``use=`` line within the section's
   configuration.  If, instead of a simple application, you're using a WSGI
   :term:`pipeline` (e.g. a ``[pipeline:main]`` section), the application
   named on the "last" element will refer to your :app:`Pyramid` application.
   If instead of a simple application or a pipeline, you're using a
   "composite" (e.g. ``[composite:main]``), refer to the documentation for
   that particular composite to understand how to make it refer to your
   :app:`Pyramid` application.  In most cases, a Pyramid application built
   from a scaffold will have a single ``[app:main]`` section in it, and this
   will be the application served.


#. Pyramid 框架会自动从 ``.ini`` 文件中查找所有日志 `logging` 相关命令
     去配置python标准日志系统。详细请参考 `logging_config` 。

#. The framework finds all :mod:`logging` related configuration in the
   ``.ini`` file and uses it to configure the Python standard library logging
   system for this application.  See :ref:`logging_config` for more
   information.

#.  Pyramid 构造器 *constructor* 的入口点是由配置中 ``use=`` 来指定的，
      系统读取配置定义里面的 键/值 对参数并把他们传入 `Pyramid` 应用。
      *constructor* 最终返回 `WSGI` 的  `router` 实例

      在  `Pyramid` 应用中， *constructor* 就是应用目录下面的一个 `package`
      的初始化函数，具体是 ``__init__.py`` 中的 ``main`` 函数。如果函数
      成功运行，他会返回一个 `router` 实例。下面是 ``__init__.py`` 内容的一
      实例。

#. The application's *constructor* named by the entry point reference or
   dotted Python name on the ``use=`` line of the section representing your
   :app:`Pyramid` application is passed the key/value parameters mentioned
   within the section in which it's defined.  The constructor is meant to
   return a :term:`router` instance, which is a :term:`WSGI` application.

   For :app:`Pyramid` applications, the constructor will be a function named
   ``main`` in the ``__init__.py`` file within the :term:`package` in which
   your application lives.  If this function succeeds, it will return a
   :app:`Pyramid` :term:`router` instance.  Here's the contents of an example
   ``__init__.py`` module:

   .. literalinclude:: MyProject/myproject/__init__.py
      :language: python
      :linenos:

      
      注意到入口点函数有两个参数，参数 ``global_config`` 接受由配置文件中
      ``[DEFAULT]`` （如果存在） 定义的键/值对组装为字典输入。而参数 
      ``**settings``    是由配置 ``[app:main]`` 定义的（除了 ``use=`` ）。
      运行命令 ``pserve`` 时会自动执行这个入口点函数  *constructor*。

   Note that the constructor function accepts a ``global_config`` argument,
   which is a dictionary of key/value pairs mentioned in the ``[DEFAULT]``
   section of an ``.ini`` file (if `[DEFAULT]
   <http://docs.pylonsproject.org/projects/pyramid/dev/narr/paste.html#defaults-section-of-a-pastedeploy-ini-file>`__
   is present).  It also accepts a ``**settings`` argument, which collects
   another set of arbitrary key/value pairs.  The arbitrary key/value pairs
   received by this function in ``**settings`` will be composed of all the
   key/value pairs that are present in the ``[app:main]`` section (except for
   the ``use=`` setting) when this function is called by when you run
   ``pserve``.

   我们通过命令生成的 ``development.ini`` 如下所示：

   Our generated ``development.ini`` file looks like so:

   .. literalinclude:: MyProject/development.ini
      :language: ini
      :linenos:
      
      在这个例子里面入口点构造 ``myproject.__init__:main`` 定义为 
      ``egg:MyProject`` （参照 `MyProject_ini` 获取更多的关于入口点是
      如何被调用的详细说明）。入口点函数接收键值参数有 ``{'pyramid.reload_templates':'true',
   'pyramid.debug_authorization':'false', 'pyramid.debug_notfound':'false',
   'pyramid.debug_routematch':'false', 'pyramid.debug_templates':'true',
   'pyramid.default_locale_name':'en'}``。这些参数的意义可以参考
   `environment_chapter` 


   In this case, the ``myproject.__init__:main`` function referred to by the
   entry point URI ``egg:MyProject`` (see :ref:`MyProject_ini` for more
   information about entry point URIs, and how they relate to callables),
   will receive the key/value pairs ``{'pyramid.reload_templates':'true',
   'pyramid.debug_authorization':'false', 'pyramid.debug_notfound':'false',
   'pyramid.debug_routematch':'false', 'pyramid.debug_templates':'true',
   'pyramid.default_locale_name':'en'}``.  See :ref:`environment_chapter` for
   the meanings of these keys.


#. ``main`` 函数首先生成了一个  :class:`~pyramid.config.Configurator` 
     对象。 ``**settings``  是作为对象的 ``settings`` 参数传入构造函数中的。

     在 ``settings`` 包含了 ``[app:main]`` 中的除了 ``use`` 外的所有配置命
     令，像 ``pyramid.reload_templates``, ``pyramid.debug_authorization``, 等
    

#. The ``main`` function first constructs a
   :class:`~pyramid.config.Configurator` instance, passing the ``settings``
   dictionary captured via the ``**settings`` kwarg as its ``settings``
   argument.

   The ``settings`` dictionary contains all the options in the ``[app:main]``
   section of our .ini file except the ``use`` option (which is internal to
   PasteDeploy) such as ``pyramid.reload_templates``,
   ``pyramid.debug_authorization``, etc.

#. ``main`` 函数接着调用之前创建 class :class:`~pyramid.config.Configurator` 
      实例的对象的各种函数。其目的就是为了完善系统  :term:`application registry`
      相关信息，来进一步配置 `Pyramid` 应用。

      

#. The ``main`` function then calls various methods on the instance of the
   class :class:`~pyramid.config.Configurator` created in the previous step.
   The intent of calling these methods is to populate an
   :term:`application registry`, which represents the :app:`Pyramid`
   configuration related to the application.

#. 最后用对象函数 :meth:`~pyramid.config.Configurator.make_wsgi_app` 来返回。
     这个函数返回一个  :term:`router` 实例。这个实例包含了由之前由各种函数
     生成的 :term:`application registry` 对象。这个router就是一个WSGI 应用。

#. The :meth:`~pyramid.config.Configurator.make_wsgi_app` method is called.
   The result is a :term:`router` instance.  The router is associated with
   the :term:`application registry` implied by the configurator previously
   populated by other methods run against the Configurator.  The router is a
   WSGI application.

#. :class:`~pyramid.events.ApplicationCreated` 事件发生（参考   :ref:`events_chapter` ）


#. A :class:`~pyramid.events.ApplicationCreated` event is emitted (see
   :ref:`events_chapter` for more information about events).

#. 如果没用错误发生，之前创建的 router 会返回到 ``pserve`` ， ``pserve``
     接收这个实例后，就生成一个  " WSGI 应用"。

#. Assuming there were no errors, the ``main`` function in ``myproject``
   returns the router instance created by
   :meth:`pyramid.config.Configurator.make_wsgi_app` back to ``pserve``.  As
   far as ``pserve`` is concerned, it is "just another WSGI application".

#. ``pserve`` 通过配置中的  ``[server:main]`` 段来启动 WSGI 服务。在
      我们的例子里面，等待服务 （ ``use =egg:waitress#main`` ）将会
      监听所有地址 ( ``host = 0.0.0.0``  ) ，监听端口为 6543 （ ``port = 6543`` ）。
      服务打印出  ``serving on 0.0.0.0:6543 view at http://127.0.0.1:6543`` 。
      然后服务应用启动，等待请求的发生。

#. ``pserve`` starts the WSGI *server* defined within the ``[server:main]``
   section.  In our case, this is the Waitress server (``use =
   egg:waitress#main``), and it will listen on all interfaces (``host =
   0.0.0.0``), on port number 6543 (``port = 6543``).  The server code itself
   is what prints ``serving on 0.0.0.0:6543 view at http://127.0.0.1:6543``.
   The server serves the application, and the application is running, waiting
   to receive requests.

.. index::
   pair: settings; deployment
   single: custom settings


.. _deployment_settings:

Deployment Settings
-------------------
部署设置
________


注意 一些附加配置参数可以由  ``**settings`` 传入到 :class:`~pyramid.config.Configurator`，
在接着系统  :term:`view callable` 中可以通过代码  ``request.registry.settings`` 
来访问。具体做法是传入配置 到 configurator 前，创建自定义的对象，插入到
``settings`` 字典类中。之后在运行时，通过 ``request.registry.settings`` 字典类
访问。

Note that an augmented version of the values passed as ``**settings`` to the
:class:`~pyramid.config.Configurator` constructor will be available in
:app:`Pyramid` :term:`view callable` code as ``request.registry.settings``.
You can create objects you wish to access later from view code, and put them
into the dictionary you pass to the configurator as ``settings``.  They will
then be present in the ``request.registry.settings`` dictionary at
application runtime.
