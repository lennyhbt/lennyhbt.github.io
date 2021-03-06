.. title: VPN 高级选项那些事
.. slug: vpn_advance_options_misc
.. date: 2014-12-24 15:08:26 UTC+08:00
.. tags: tech, vpn
.. category: tech
.. link:
.. description:
.. type: text
.. author: lennyh

一、VPN高级选项
=====================

VPN高级选项有哪些，都是什么意思
----------------------------------

* DNS 搜索域
    These are for the mechanism for going from a machine name to a Fully Qualified Domain Name.

    DNS searches can only look at a Fully Qualified Domain Name, such as mymachine.example.com. But, it's a pain to type out mymachine.example.com, you want to be able to just type mymachine.

    Using Search Domains is the mechanism to do this. If you type a name that does not end with a period, it knows it needs to add the search domains for the lookup. So, lets say your Search Domains list was: example.org, example.com

    mymachine

    would try first mymachine.example.org, not find it, then try mymachine.example.com, found it, now done.

    mymachine.example.com

    would try mymachine.example.com.example.org (remember, it doesn't end with a period, still adds domains), fail, then mymachine.example.com.example.com, not find it, fall back to mymachine.example.com, found it, now done

    mymachine.example.com. Ends with a period, no searching, just do mymachine.example.com

    Soooo.....

    If you have your own DNS domain such as example.com, put it there. If not, ignore it. It really is more corporate than a home setting.

    参考 `What is the “Search Domains” field for in the tcp/ip DNS settings control panel/preference pane for?> <http://superuser.com/questions/184361/what-is-the-search-domains-field-for-in-the-tcp-ip-dns-settings-control-panel>`_

* DNS 服务器
    域名系统（英文：Domain Name System，縮寫：DNS）是因特网的一项服务。它作为将域名和IP地址相互映射的一个分布式数据库，能够使人更方便的访问互联网。DNS 使用TCP和UDP端口53。当前，对于每一级域名长度的限制是63个字符，域名总长度则不能超过253个字符。

    参考 `域名系统 <http://zh.wikipedia.org/zh/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F>`_

* 转发路线
    即转发路由，因DNS服务器不提供任何服务，因此所有的请求都需要经过转发才能到达可以响应请求的服务器，转发路线即配置发往哪些地址请求的需经过VPN转发。

为什么需要这些选项
----------------------

这些选项其实只是一些基础的网络参数，因此所有的网络连接（包括VPN）都需要这些选项。但是并不是所有的网络连接都需要手动配置这些参数。那么为什么VPN更加的需要配置这些参数呢？

VPN是一种常用于连接中、大型企业或团体与团体间的私人网络的通讯方法。虚拟私人网络的讯息透过公用的网络架构（例如：互联网）来传送内联网的网络讯息。这种技术可以用不安全的网络（例如：互联网）來傳送可靠、安全的訊息。[摘自 `危机百科`_ ]

DNS请求呢？在未配置VPN之前，我们使用的是不安全网络上的DNS服务器，如果连接到VPN之后我们仍然连接不安全网络上的DNS服务器，如何保证我们的数据安全？
DNS搜索域是一个方便使用的选项。

路由则指定了哪些数据是需要VPN网络来保护的，如果不指定，或者系统中所有的流量都经过VPN，但是VPN连接到的私有网络并不能提供不安全网络中所有的网络请求；或者
系统中所有的流量都不经过VPN服务器，连接VPN干嘛，当摆设吗？

由此可见，这些网络高级选项在VPN的配置中还是很有必要的。


二、关于connman
========================
使用connman管理系统网络连接的例子并不多，网络上相关的资料也很少。
万幸的是，connman自带的文档(doc目录)大概可以把connman的设计原则和使用方法解释清除了。
以下内容及为参考该文档及源代码以及本人的推测得来的，不一定准确。

connman是如何管理所有连接的
------------------------------
::

    src/provider.c  -- 管理connman中每一个（不是每一种）连接，保存连接。
    src/service.c   -- 管理连接服务。
    src/task.c  -- connman中对连接的代码，负责创建与维护真正的连接进程。

这几个代码文件大概实现了connman连接管理的框架，但是新建连接后还需要设置很多的网络参数，
等等，正是这一部分使得connman显得更加的复杂。

connman是如何管理网络参数的（路由、DNS等）
--------------------------------------------
connman中对网络参数的管理是基于连接的，即每个连接都有不同的网络参数配置，该连接生效时connman会根据
连接属性更新系统的网络参数。

connman封装了很多对系统网络参数修改的API，如下列举部分::

    src/inet.c  -- 实现对系统路由的配置
    src/ipconfig.c  -- 实现对系统地址的配置
    src/resolver.c  -- 实现对系统DNS的配置，connman有选项支持dns代理

.. note::
    connman-vpn与上述描述并不完全一致。
    当然，vpn连接也是由connman_task创建具体的任务来连接的，但是。
    vpn/vpn-manager.c提供新建/删除VPN连接的功能(create/remove/get_connections...)。
    vpn/vpn-provider.c提供了vpn连接/断开功能(do_connect/do_disconnect...)。
    vpn连接建立/删除时会发送ConnectionAdded/ConnectionRemoved信号，
    vpn连接时会发送PropertyChanged信号。
    connman的vpn插件会监听这些信号，在新建/删除vpn连接时会在connman进程中建立该连接的provider及service。
    connman监听到PropertyChanged信号时会根据属性设置系统当前的网络参数（dns等）。

推荐文档阅读顺序：
vpn-overview.txt -> vpn-manager-api.txt -> vpn-connection-api.txt


四、Qt 、QML and D-Bus
===========================

Connman是以daemon进程在系统后台运行的，要访问Connman提供的服务，只能通过进程间通信类似的机制。
事实上Connman提供的服务都是以D-Bus方法即信号作为API接口的。

例如，新建/删除VPN、连接VPN的接口如下：

.. code-block:: c

    static DBusMessage *create(DBusConnection *conn, DBusMessage *msg, void *data);
    static DBusMessage *remove(DBusConnection *conn, DBusMessage *msg, void *data);
    static DBusMessage *do_connect(DBusConnection *conn, DBusMessage *msg, void *data);

Qt对D-Bus的支持
-------------------
Qt对D-Bus的支持算是基本完善，该有的都可以有，不该有的可能会可以有。（：D）

可以通过Qt中D-Bus相关的库函数创建D-Bus服务，或者使用别人提供的服务。对发送接收数据类型的支持也比较完整，不仅能够
收发基本的整数、字符串等，复杂的字典、数组等自然也不在话下。

但是Qt对DBUS_TYPE_STRUCT的支持需要稍多做一些工作，下面的章节会有介绍。

QML对D-Bus的支持
---------------------

很遗憾，QML原生并不支持D-Bus，但是可以通过两种变通的途径使用。
第一是，在C++代码中封装调用D-Bus的接口，并注册到QML中。
第二种，是采用非QT官方的插件，实现，例如： `Nemo Mobile D-Bus QML Plugin <https://github.com/nemomobile/nemo-qml-plugin-dbus>`_

好吧，其实是一种，第二种其实同样是C++代码中封装了调用D-Bus的接口，但是除此之外，还有什么办法可以扩展QML不支持的功能吗？

Qt对D-Bus中DBUS_TYPE_STRUCT的支持
---------------------------------------

Qt有自己的类型系统，不知是该庆幸还是该懊恼。
Qt的类型系统极大的丰富了我们的精神文化生活，噢不，是极大的方便了我们的开发，QVariant，信号/槽（QObject）等等。
但是这样一来我们自己定义的类型却无法使用这些方便的特性，而且Qt D-Bus也不支持自定义类型的发送与接收。

幸运的是，上帝在关上这扇门的时候悄悄给我们开了一扇窗，我们可以将自己定义的类型注册到Qt的元类型系统中去，
这样我们自己定义的类型也可以使用Qt提供的很多方便的特性了，最重要的是我们自定义的结构也可以通过Qt的D-Bus接口发送与接收了。

创建方法在此不表，无非是在适当的地方增加几次调用::

    Q_DECLARE_METATYPE(Type)；
    int qRegisterMetaType(const char * typeName)；
    int qDBusRegisterMetaType()；

详情参看如下链接：

* `创建自定义Qt类型 <http://qtdocs.sourceforge.net/index.php/%E5%88%9B%E5%BB%BA%E8%87%AA%E5%AE%9A%E4%B9%89Qt%E7%B1%BB%E5%9E%8B>`_
* `Problems with marshalling a struct to Qt/DBus <http://www.qtcentre.org/threads/26871-Problems-with-marshalling-a-struct-to-Qt-DBus>`_


五、Linux 连接管理
========================

都有哪些连接管理实现
----------------------
* Android：`ConnectivityManager <http://developer.android.com/reference/android/net/ConnectivityManager.html>`_

* `NetworkManager <https://wiki.gnome.org/Projects/NetworkManager>`_
    NetworkManager is a set of co-operative tools that make networking simple and straightforward. Whether Wi-Fi, wired, bond, bridge, 3G, or Bluetooth, NetworkManager allows you to quickly move from one network to another: once a network has been configured and joined, it can be detected and re-joined automatically the next time its available.

* `ConnMan <https://01.org/zh/connman?langredirect=1>`_
    ConnMan is a daemon for managing Internet connections within embedded device and integrates a vast range of communication features usually split between many daemons such as DHCP, DNS and NTP. The result of this consolidation is low memory consumption with a fast, coherent, synchronized reaction to changing network conditions.

为什么需要连接管理
-----------------------
几乎所有的现代操作系统都有统一的连接管理，这是为什么呢？
其实这个问题我也不知道。所以，下面的内容纯属揣测，如有不对恳请指正。

大概是有两个原因吧，我想。
一是便于用户的配置，试想，用户连接上一个新的网络（有线、无线、VPN等）后，要手动的去修改DNS、路由、地址等信息，
肯定是不可原谅的，或者进一步，需要在不同的位置分别通过不用的程序去配置不同的网络参数，少改了一项网络可就不正常了哦。

二是便于网络的管理，如果每种连接自己管自己的网络配置，可是这些配置的生效可是在一个系统上的，于是每个程序都去修改
DNS配置，路由，地址等信息，你确保不会改乱？

其实反观其他子系统，声音肯定要在所有要播放/录制声音的程序后面有一个daemon来负责系统的混音及播放工作，不可能每个程序各播各的，你肯定不原因听到那种声音的。
显示子系统不可能是每个想要在屏幕上显示东西的程序自己向屏幕上写吧，这样我显示了一个窗口，你显示了一个通知，我有显示了一个文档，你确定用户能够看得请？所以还是需要有显示管理器在后面跑的。

同理，系统的网络配置大家一起改，你确定不会改乱？
这大概是一个趋势吧，只有一种或者两种网络连接的时候，我可以随便改，要是系统有很多种连接类型，可就不能胡来了。


.. _危机百科:  http://zh.wikipedia.org/zh/%E8%99%9B%E6%93%AC%E7%A7%81%E4%BA%BA%E7%B6%B2%E8%B7%AF>
