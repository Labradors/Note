---
title: TigaseV7.1.0发布
date: 2016-11-20 10:32:45
tags: 翻译
categories: 后端

---

> Note:TigaseV7.1.0发布，原文链接：[Tigase v7.1.0 announcement](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#tigase710)。
>
> 近期一直在忙看英语文档，前面欠的四五篇博客，只有过段时间再补上了。Tigase的中文文档实在太少，很多写的也不太详细。自己慢慢看也能看懂，只是看了这篇上一篇又不太记得。记录一下。以后能把英文文档完全当中文看了，就不用记录了。<!--more-->

**欢迎查看Tigase管理员指南**,为了给你提供更安全，更精简，更好的`XMPP`服务，我们一直在改进和实现一些Tigase服务器程序的新功能。

我们分享了一些新功能，组件和大量的补丁。请注意，不是所有的补丁都可以查看，因为有一些补丁可能包含客户权限问题。

## 重大改变

`Tigase V7.1.0`已经从代码和结构上有重大的改变。为了继续使用`Tigase`，一些改变可能需要对你的系统进行相应的改变。具体内容请继续看：

### HTTP组件重命名

我们重命名了`HTTP`组建，如果你仍然在*init.properties*配置文件中使用过时的配置`tigase.rest.RestMessageReciever`，请更新组建名为：

```java
tigase.http.HttpMessageReceiver
```

### 需要`jdk8`

因为**Oracle**已经放弃对`jdk7`的支持，所以我们不得不升级为`jdk8`。此外，一些新特性和一些补丁也需要`jdk8` 版本的支持。请更新你的`jdk`版本。

###  数据库模式改变

Tigase已经对数据库模式进行了更改，最近的版本主要是*database schema v7.1*和*pubsub schema v3.2.0*。如果你想从以前的数据库版本升级至当前`v7.1`版本，我们推荐你阅读[这一节](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#v710notice)准备新的安装。

### `Presence`插件拆分

`Presence.java`这个处理`presence`相关操作的插件已经被分离出来。`PresenceAbstract.java`一般处理大多数与`presence`相关的方法。他同时也被以下两个插件所使用。

- `PresenceSubscription.java`负责用户状态的更新，如花名册处理等。
- `PresenceState.java`负责一个新的连接，登录的初始化操作。

## 新特性与组建

### 新的 HTTP API

Tigase具有`http api`,所有可以进行`web`端的聊天，管理员可以改变设置，管理用户，甚至可以从浏览器窗口中编写和运行脚本。此外，可以通过`rest api`接口来运行自定义的脚本和命令。我们正在计划改进这个ui的界面设计。使其越来越丰富。让用户享受这个实时的通信和漂亮的界面。

### 新的管理员接口

Tigase现在内置了`Web XMPP`客户端。你可以从[http://yourhost.com:8080/ui/](http://yourhost.com:8080/ui/).这个链接访问。关于更多细节，请查看Admin UI guide。

### 添加XEP-0334支持

XEP-0334 现在已经被支持，具体详情，请查看[这一节](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#nonBodyElements)。也可以查看[XEP-0334: Message Processing Hints](http://xmpp.org/extensions/xep-0334.html),后面专门写一篇博文来描述这些协议的具体内容。

### 核心Kernel Bean配置器改进

为`Bean`属性添加别名来使用高级配置。

```x
component/bean-name/property=value
```

下面的方法更容易使用:

```xm
component/property=value
```

### 支持XEP-0352

客户端状态指示可以默认在`XMPP SERVER`中可以使用，[详情](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#sessManMobileOpts)

### 一个证书适用多个虚拟主机

Tigase现在允许为每一个虚拟主机验证启用通配符的服务器端验证。[详情](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#sessManMobileOpts)

### 群聊最大用户数量限制

管理员可设置群聊的最大限制人数。详情可查看[MUC Room Configuration](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#mucRoomConfig).

### Rest API支持

Tigase现在允许为http提供`rest api`。可查看[文档](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#tigase_http_api)。

### 空的Nicknames

Tigase可以使用空的昵称，查看[this](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#emptyNicks) 。

### 离线消息限制

Tigase允许使用AMP来更改离线消息限制。[Documentation here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#offlineMessageLimits).

### 离线消息保存

Tigase实现了一种新的方式去实现离线消息保存。它不会替换原有的标准脱机消息。但可以使用另外一种方式。[Documentation here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#offlineMessageSink).

### 白名单设置

组件可以添加到白名单，它将被共享到集群中所有的主机。[#3244](https://projects.tigase.org/issues/3244)

### Tigase邮件拓展

Tigase邮件拓展现在被内置到`xmpp server`中，这个拓展允许当监视器被触发时，监视器可以从一个特殊的email地址传递到另外一个email地址。详情[monitor mailer section](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#monitorMailer).

### EventBus实现

Tigase有一个简单的Pubsub组件去调用EventBus来报告任务和触发器。More details are available [Here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#eventBus).

### XEP-0191 Blocking Command支持

Tigase已经添加了阻塞命令支持。all functions of [XEP-0191](http://xmpp.org/extensnions/xep-0191/html) should be implemented. See [Admin Guide](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#blockingCommand) for details.

### 流管理添加设置超时功能

最大的流超时时间和默认流超时时间可以在`init.properties`中设置。Details of these two settings can be found [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#streamResumptiontimeout).

### JVM默认配置更新

安装`Tigase Server`后，默认的JVM选项已经被添加到tigase.conf，如下：

```java
PRODUCTION_HEAP_SETTINGS=" -Xms5G -Xmx5G " # heap memory settings must be adjusted on per deployment-base!
JAVA_OPTIONS="${GC} ${EX} ${ENC} ${DRV} ${JMX_REMOTE_IP} -server ${PRODUCTION_HEAP_SETTINGS} -XX:MaxDirectMemorySize=128m "
```

我们推荐为每一个主机调整堆内存设置。see [our Documentation](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#jvm_settings).

### Java垃圾回收机制已经改进

在经过详细的调查和重大测试之后，我们调整了java垃圾回收机制，以防止内存使用过高。[#3248](https://projects.tigase.org/issues/3248)

For more information about JVM defaults and changes to settings, see [our Documentation](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#jvm_settings).

### 新的Rest API 添加了jid登录时间

GetUserInfo已经添加了最近的登录和登出时间。See [this section](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#getUserInfoREST) for full details.

### 新的init.properties属性

`--ws-allow-unmasked-frames=false`和`not force-close the=true`允许通过`Websocket`来发送未屏蔽的帧。RFC 6455 指出，所有的客户端必须屏蔽通过`WebSocket`连接来发送到服务端的帧。如果未屏蔽帧被发送，没有经过任何加密处理，服务器端必须关掉这个连接。然而，一些客户端也许不支持屏蔽帧。或者为来开发目的，你想绕过这个安全措施。即可这样使用。

`--vhost-disable-dns-check=true`设置虚拟主机DNS检查为不检查，当一个虚拟主机被创建，Tigase将自动检查SRV记录和正确的DNS设置以确保用户可以连接。然而，如果这些校验失败，你将不能保存这些改变。这些设置允许你绕过这些检查。

### 连接看门狗

看门狗属性允许检查一些过时的连接，当这些连接出现问题时，及时的切断这些连接。More details [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#watchdog).

### Web设置界面现在具有受限访问

可以通过访问[http://yourserver.com/8080/setup/](http://yourserver.com/8080/setup/)来查看web安装器设置界面。登录需要admin的jid或者在init配置文件中指定。See the [Web Installer](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#webinstall) section for default settings. See [Component Properties](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#httpCompProp) section for details on the new property.

### 可配置离线消息存储

当管理员想存储离线消息时，可以通过离线消息拦截器和控制器去存储她们想要存储的消息。See [more details here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#offlineMessageReceipts).

### 账户注册限制

为了保护Tigase Server不被dos攻击，我们已经实现了每分钟的账号注册限制。See [this link](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#accountRegLimit) for configuration settings.

### 对请求不可用资源的的数据包采用静默忽略

现在你可以将Tigase忽略数据包设置为不可用资源，以避免消耗流量。Learn how [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#silentIgnore).

### 集群连接改进

集群现在使用CLUSTER优先级运行。给予数据包高于HIGH的状态，否则会在大量断开连接时间引起问题。新的配置做了如下改变：

- 在*init.property*中设置CLUSTER数据包的连接数。

```xml
cl-comp/cluster-sys-connections-per-node[I]=2
```

- 一个新的类实现一个新的连接接口，但是使用旧的连接机制。并且，任何连接都可以发送命令。

```x
cl-comp/connection-selector=tigase.cluster.ClusterConnectionSelectorOld
```

我是没看懂。

### 集群连接测试实现

看门狗默认被添加到测试集群中，看门狗默认每30秒发送一个ping到集群中，以检查在三分钟内有没有收到一个ping的响应。如果没有，这个集群连接将被销毁。全局的看门狗设置将不会影响集群的功能。

### 集群映射实现

Tigase可以通过一个api来生成集群的映射。See the [development guide](http://docs.tigase.org/tigase-server/snapshot/Development_Guide/html/#clusterMapInterface) for a description of the API.

### 新的许可程序

With the release of Tigase XMPP server v7.1.0, our licensing procedures have changed. For more information about how to obtain, retain, and install your license, please see [this section](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#licenseserver).

### 消息存档支持非消息体元素

消息存档现在可以存储没有消体的消息。this option is explained in [this section](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#nonBodyStore).

### 直接清除消息存档数据

可以根据时间和自动过期时间来自动或者手动清理这些统一存档或者消息存档数据。Information on configuring this is available [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#maPurging).

### 服务器统计信息

Tigase XMPP Server统计信息已经被拓展，它可以会话关闭时打印。或者以正常的方式获得。注意，对比以前的版本，统计信息有所改变。有一些不同的格式。See [the Statistics Description](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#statsticsDescription) section of the Administration guide for all current server statistics.

### 强制重定向

可以通过` force-redirect-to`去从一个端口重定向到另外的端口。[Details here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#_enforcing_redirection).

### 改进认证方式

SASL-SCRAM和 SCRAM-SHA-1 - SCRAM-SHA-1-PLUS被支持。

### 双IP安装

Tigase现在有一个双IP设置，现在可以使用单独的内部和外部IP，并使用DNS解析器进行连接重定向。 Setup instructions are [Located here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#_configuring_hostnames).

### 错误计数

可以从服务器统计中获取，错误的信息和错误数量。This feature is explained in more detail [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#errorCounting).

### 新的数据库计数器

三个新的统计信息被添加到basic-conf中，以监视数据库的稳定性。以及xmpp需要重新连接的数据库的频率。The list of new statistics are listed [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#repo-factoryStatistics).

### 集群统计

集群统计信息已经添加到cl-comp中，显示连接的集群的节点数。显示为INFO级别统计。

### 新的文档结构

就是文档改变。

### 最后可用状态的xml信息可以完整的保存到数据库

现在可以把最后的xml信息保存到数据库,以及整个存在节之前的时间戳保存到存储库。More information is available [here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#storeFullXMLLastPresence).

### 自动订阅设置

Tigase支持启用自动存在订阅和花名册授权。有关这些设置的详细信息，请选中自动订阅部分。check the [Automatic Subscriptions](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#autoSub) section.

### 默认值更改

为了提高性能，Tigase Server已经改进了默认的配置。包括如下：

- 增加与CPU相匹配的数据库连接。
- 默认的线程池已经增加，以提升性能。
- 增加线程池计数器。
- 改进java默认设置。
- 改进java垃圾回收机制。

## 一些小的更改和动作更改

- 旧的监视器组建已经过时和关闭。
- JTDS MS SQL Server驱动已经更新到v1.3.1。
- tigase-utils 和 tigase-xmltools 现在已经包含在tigase-server。
- Tigase Kernal已经被更新和改进。
- tigase.stats.CounterDataFileLogger现在已经包含时间戳。
- Javadoc不再安装时生成，因为已经包含在发行版中。
- 管理员节点连接事件已经被改进。
- ServerConnectionManager已经被完全弃用。
- 日志中已经包含重连信息。
- [#163](https://projects.tigase.org/issues/163) [XEP-0012](http://xmpp.org/extensions/xep-0012.html) User LastActivity implemented
- [#593](https://projects.tigase.org/issues/593) [XEP-0202 Entity Time](http://xmpp.org/extensions/xep-0202.html) has been implemented.
- [#788](https://projects.tigase.org/issues/788) End User Session from [XEP-0133 Service Administration](http://xmpp.org/extensions/xep-0133.html) implemented.
- *#811* 插件api可包含更多的xml信息。
- [#813](https://projects.tigase.org/issues/813) 集群节点默认的连接数设置为5，CLUSTER流量界别设置为2
- [#1436](https://projects.tigase.org/issues/1436) ClusterConnectionManager现在每30秒发送ping数据包，以检查活动集群连接的状态。
- [#1449](https://projects.tigase.org/issues/1449) 可在OSGI模式下运行监视器。
- [#1601](https://projects.tigase.org/issues/1601) XMPPPresenceUpdateProcessorIFC接口被删除，替换为Eventbus专有线程池模式。
- [#1783](https://projects.tigase.org/issues/1783) 集群节点的连接于重连已经改进，集群现在接收一个'isclusterready？'连接前查询，避免同步问题。此外，还添加了超时的问题。See [Properties guide](http://docs.tigase.org/tigase-server/snapshot/Properties_Guide/html/#clusterPortDelayListening) for more details。

### 实在太多了，不太重要的就直接英语吧。

- [#2426](https://projects.tigase.org/issues/2426) Support for [XEP-0334](http://xmpp.org/extensions/xep-0334.html) has been added.
- [# 2530](https://projects.tigase.org/issues/2530) RosterFlat implementation now allows for a full element to be injected into presence stanzas instead of just a custom status.
- [#2561](https://projects.tigase.org/issues/2561) & [#85](https://projects.tigase.org/issues/85) Offline messages now consider sessions without presence & resources negative priority in delivery logic.
- [#2596](https://projects.tigase.org/issues/2596) Delivery errors are no longer run through preprocessors.
- [#2823](https://projects.tigase.org/issues/2823) staticStr element method now implemented.
- [#2835](https://projects.tigase.org/issues/2835) Allowing of setPermissions on incoming packets before they are processed by plugins.
- [#2903](https://projects.tigase.org/issues/2903) see-other-host has new option to make it configurable on a per vhost basis.
- [#3034](https://projects.tigase.org/issues/3034) Improved handling of data types and primitives within Tigase.
- \#3173 Stanzas with unescaped XML special characters are now ignored instead of sending a force-close of connection to sender.
- [#3180](https://projects.tigase.org/issues/3180) Protected access to JDBC repository now enabled.
- [#3230](https://projects.tigase.org/issues/3230) Verification added to check against CUSTOM domain rules when submitted.
- \#3258 Retrieval of PubSub/PEP based avatars using REST API now supported. [Command URLs here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#avatarRetrievalRequests).
- \#3282 VCard4 support added along with VCardTemp compatibility and integration.
- [#3285](https://projects.tigase.org/issues/3285) Stream Management changed to fully support XEP-0203.
- [#3330](https://projects.tigase.org/issues/3330) Error for adding users already in db now returns Error 409 with User exists.
- \#3364 Clustering support has been re-factored to remove duplicate nodeConnected and nodeDisconnected methods.
- \#3463 offline-roster-last-seen feature as a part of presence probe is now disabled by default.
- [#3496](https://projects.tigase.org/issues/3496) TigUserLogout has been improved to use sha1_user_id = sha1(lower(_user_id)) instead of "_user_id".
- [#3511](https://projects.tigase.org/issues/3511) Stream closing mechanism in SessionManager, new STREAM_CLOSED command has been added to organize shutdown of XMPP streams.
- \#3569 Fixed error occuring when attempting to remove offline users from roster.
- \#3609 Added new configuration option for BOSH to disable hostname attribute. [Details here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#tip_1_bosh_in_cluster_mode_without_load_balancer).
- [#3670](https://projects.tigase.org/issues/3670) Hardened mode now uses long DH keys (2048) by default.
- [#3849](https://projects.tigase.org/issues/3849) New Roster size limit configurable setting. See info [Here](http://docs.tigase.org/tigase-server/snapshot/Administration_Guide/html/#rosterLimit).
- [#3872](https://projects.tigase.org/issues/3872) PostgreSQL driver updated to v9.4.
- \#3892 PEP plugin now supports processing of [http://jabber.org/protocol/pubsub#owner](http://jabber.org/protocol/pubsub#owner).
- [#3908](https://projects.tigase.org/issues/3908) Logs now print whether components or plugins are depreciated, and recommend configuration settings changes.
- [#3937](https://projects.tigase.org/issues/3937) Windows setup given one-click solution to file initialization.
- [#3945](https://projects.tigase.org/issues/3945) SSLContextContainer has been replaced with JDKv8 extension version now known as SNISSLContextContainer adding support for SNI of SSL/TLS.
- [#3948](https://projects.tigase.org/issues/3948) Tigase PubSub now responds to disco#info requests in line with [XEP-0060 - Discover Node Metadata](http://xmpp.org/extensions/xep-0060.html#entity-metadata).
- [#3950](https://projects.tigase.org/issues/3950) MongoDB driver updated to v2.14.1.
- \#3985 Improved disco#info to return extended results for MUC component as per [XEP-0128 Service Discovery Extensions](http://xmpp.org/extensions/xep-0128.html).
- \#3986 Added new index for tig_nodes collection in MongoDB Databases.
- [#4003](https://projects.tigase.org/issues/4003) VisualVM and statistics gathering has been improved and refactored for a lower footprint, and a number of new windows and features for program monitoring.
- \#4020 SeeOtherHostSualIP implementation has been fixed to support MongoDB.
- [#4120](https://projects.tigase.org/issues/4120) Duplication of messages in offline storage when Message and OfflineMessage processors are used has been resolved.
- \#4127 Database reconnection procedure improved, increased `SchuduledExecutorService` threads to 2, `ClusterConnectionManager` threads to 4, and added timeout parameter to timeout task.
- \#4162 xml:lang attribute is now supported in Tigase MUC component.
- \#4248 Changed ErrorCounter from XMPPProcessorIFC to XMPPPacketFilterIfc for more accurate functionality.
- [#4254](https://projects.tigase.org/issues/4254) Fixed <code>JabberIqPrivacy</code> to properly compare JID’s instead of strings, improved partial JID matching, and added tests to add non-case sensitive comparison in MA/UA.
- [#4256](https://projects.tigase.org/issues/4256) Reduced Statistics memory usage by interning statistics labels and changing data types.
- [#4356](https://projects.tigase.org/issues/4356) Message Archive component has been converted to kernel and beans.
- [#4352](https://projects.tigase.org/issues/4352) Websocket implementation has been changed to properly parse HTTP headers with omitted spaces for HTTP 1.1 protocol.
- [#4358](https://projects.tigase.org/issues/4358) Several methods have been renamed or removed to prepare for v7.2.0 Kernel setup.
- \#4385 Minor tweak to web-installer to skip unnecessary showing of blank init.properties file & enabled post-setup editing and saving.



## 补丁

- [#8](https://projects.tigase.org/issues/8) XML parser no longer passes malformed XML statements to server.
- [#1396](https://projects.tigase.org/issues/1396) & [#663](https://projects.tigase.org/issues/663) User roster behaves correctly. Tigase now waits for user authorization before users are added to a Roster.
- [#1488](https://projects.tigase.org/issues/1488) NPE in ad-hoc for managing external components fixed.
- [#1602](https://projects.tigase.org/issues/1602) Minor optimization in MessageCarbons with new functions added to XMPPResourceConnection.
- [#2003](https://projects.tigase.org/issues/2003) Fixed bug with C2S streams where server would not always overwrite from attribute with full JID in subcription-related presence stanzas.
- [#2118](https://projects.tigase.org/issues/2118) Username modification bugfix. Tigase now returns "" for blank usernames instead of string after a username has been made blank.
- [#2859](https://projects.tigase.org/issues/2859) & [#2997](https://projects.tigase.org/issues/2997) STARTTLS stream error on SSL sockets fixed.
- [#2860](https://projects.tigase.org/issues/2860) Fixed issue with SSL socket client certificate not working.
- [#2877](https://projects.tigase.org/issues/2877) Fixed issue in Message Carbons if message contains AMP payload.
- [#3034](https://projects.tigase.org/issues/3034) Streamlined primitive and Object array handling.
- [#3067](https://projects.tigase.org/issues/3067) Fixed Bug where if duplicate commands were sent to MS SQLServer a race condition would occur.
- [#3075](https://projects.tigase.org/issues/3075) Fixed error when compiling Tigase in Red Hat Enterprise Linux v6.
- [#3080](https://projects.tigase.org/issues/3080) --net-buff-high-throughput now parses integers properly. Setting no longer reverts to default when new values are set.
- [#3126](https://projects.tigase.org/issues/3126) Calculation of percentage of heap memory used in Statistics now selects proper heap.
- [#3131](https://projects.tigase.org/issues/3131) Fixed messages with AMP payload bound for plugins getting redirected to AMP for processing.
- [#3150](https://projects.tigase.org/issues/3150) Default Log level changed for certain records. All log entries with skipping admin script now have log level FINEST instead of CONFIG
- [#3158](https://projects.tigase.org/issues/3158) Fixed issue with OSGi not reporting proper version, and PubSub errors in OSGi mode.
- [#3159](https://projects.tigase.org/issues/3159) User Privacy lists now activate properly and does not wait for presence stanza to filter packets.
- [#3164](https://projects.tigase.org/issues/3164) Fixed NPE in StreamManagementIOProcessor when <a/> is processed after connection is closed.
- [#3166](https://projects.tigase.org/issues/3166) NPE in SessionManager checking SSL null connections fixed.
- [#3181](https://projects.tigase.org/issues/3181) S2S connection multiplexing now has consistent behavior.
- [#3194](https://projects.tigase.org/issues/3194) Fixed issue with single long lasting HTTP connection blocking other HTTP requests. Default timeout set to 4 threads after 60 seconds.
- [#3200](https://projects.tigase.org/issues/3200) Implemented a faster way to close stale connections using MS SQL server, reducing calm down time after large user disconnects.
- \#3203 Correct presence status shows for contacts if authorization was accepted while user was offline.
- [#3223](https://projects.tigase.org/issues/3223) GetUserInfo ad-hoc command no longer omits information about local sessions when a remote session is active.
- \#3226 Fixed NPE & argument type mismatch in Pubsub.
- [#3245](https://projects.tigase.org/issues/3245) Fixed ClassCastException when Websocket is configured to use SSL.
- [#3249](https://projects.tigase.org/issues/3249) JabberIQVersion plugin now returns proper client information when requested from self.
- [#3259](https://projects.tigase.org/issues/3259) Websocket no longer loops when receiving stanzas between 32767 and 65535 bytes in size.
- [#3261](https://projects.tigase.org/issues/3261) Fixed issue with duplicate disco#info responses.
- [#3274](https://projects.tigase.org/issues/3274) NPE when removing roster nickname fixed.
- [#3307](https://projects.tigase.org/issues/3307) Rosters are no longer re-saved when a user logs in and roster is read resulting in a performance boost.
- [#3328](https://projects.tigase.org/issues/3328) Presence processing by PEP plugin optimized.
- [#3336](https://projects.tigase.org/issues/3336) Fixed issues with reloading vhosts in trusted after configuration change.
- [#3337](https://projects.tigase.org/issues/3337) tls-jdk-nss-bug-workaround-active is now disabled by default. This fix is disabled by default which may impact older OpenSSL versions that may no longer be supported. You may enable this using an init.properties setting.
- \#3341 IQ Packet processing changed for packets sent to bare JID in Cluster mode.
- [#3372](https://projects.tigase.org/issues/3372) Fixed NPE when presence was re-broadcasted to users who did not exit server gracefully.
- [#3374](https://projects.tigase.org/issues/3374) PubSub Schema changed to be more compatible with MS SQL.
- [#3375](https://projects.tigase.org/issues/3375) Users removed VIA REST commands are now disconnected immediately.
- [#3386](https://projects.tigase.org/issues/3386) Fixed AMP logic to avoid querying for (default) Privacy list if user does not exist.
- \#3389 Fixed issue of sending packets to connections that were closed, but connection write lock had not been acquired.
- [#3401](https://projects.tigase.org/issues/3401) Multiple issues fixed with Tigase.IM web client.
- [#3422](https://projects.tigase.org/issues/3422) UTC Timestamps now enforced inside cluster_nodes table.
- \#3440 Fixed WebSocket error 12030 showing unexpectedly.
- [#3446](https://projects.tigase.org/issues/3446) Fixed Installer configuring MUC incorrectly.
- \#3449 Wrapper.conf updated with current library folder for windows Service wrapper.
- [#3453](https://projects.tigase.org/issues/3453) Fixed NPE when using comparator when sorting messages.
- \#3485 Fixed JDBCMsgRepository inserting duplicate user JID into table while using AMP.
- [#3489](https://projects.tigase.org/issues/3489) Various fixes to Tigase test suite. Fixed race condition from XMPPSession conflicts when new sessions and closing session events happen at the same time.
- [#3495](https://projects.tigase.org/issues/3495) Fixed messages being duplicated by message carbons.
- \#3499 Various fixes to AMP component.
- \#3530 Fixed null cert chain error when connecting to other servers using S2S connection with StartTLS.
- \#3550 Fixed NPE in sess-man when trying to delete all user information using Pidgin or Psi.
- [#3556](https://projects.tigase.org/issues/3556) JavaDoc updated to include documentation for xmltools, tigase-extras, and tigase-util packages.
- [#3559](https://projects.tigase.org/issues/3559) Fixed Web admin UI not updating Cluster node when it id disconnected.
- [#3579](http://projects.tigase.org/issues/3579) Fixed NPE in SimpleParser.
- [#3580](http://projects.tigase.org/issues/3580) Replaced misleading feature not implemented error when SM attempts to put a packet to processor and queue is full.
- \#3598 Fixed error in removing users from blocked list.
- \#3599 Fixed FlexibleOfflineMessages not being delivered to connection due to lack of explicit connection addressing.
- \#3612 Fixed issue when processing packets sent to full JID in cluster mode when user is connected to more than one cluster node at once.
- \#3619 Fixed issue with non-presistent contacts being unable to be added to roster.
- \#3649 Changed privacy list processing to always allow communication between XMPP connections with the same BareJID.
- [#3655](https://projects.tigase.org/issues/3655) Increased max loop in infinity loop detection logic to 100000 in order to aid larger transfers.
- \#3656 Add option to BOSH output command without a timer task to avoid generation of packets to closed connections.
- \#3686 XHTML-IM parser has been fixed, restoring [XEP-0071](http://xmpp.org/extensions/xep-0071.html) functionality.
- [#3688](https://projects.tigase.org/issues/3688) Issues with Eventbus in cluster mode fixed.
- [#3689](https://projects.tigase.org/issues/3689) Avoid using sender address when packets are returned from Cluster Manager using stream management.
- \#3717 Support added to store messages without <body/> element if storage method other than <body/> is used. Support also added for JAXMPP to retrieve whole element from Message Archiving instead of only <body>.
- \#3718 Removed DISCONNECTING! debug stanza from AbstractWebSocketConnector.java that was causing NPE when user fails authentication in WebSocket.
- [#3753](https://projects.tigase.org/issues/3753) Fixed NPE when using Blocking command.
- [#3775](https://projects.tigase.org/issues/3775) Fixed ThreadExceptionHandler error in Tigase mailer.
- [#3781](https://projects.tigase.org/issues/3781) Fixed issue with sending C2S message "The user connection is no longer active".
- [#3800](https://projects.tigase.org/issues/3800) Changed Jenkins to always pull latest binaries from repositories. Windows wrapper changed to use wildcards to load /jars folder.
- \#3848 Changes made to JDBCMessageArchiveRepository to fix potential MySQL deadlocks when adding entries to repository.
- \#3902 Fixed issue where wss:// connections were closed after 3 minutes of inactivity.
- \#3910 Fixed NPE in SessionManager when session was closed during execution of everyMinute method.
- \#3911 Fixed load distribution error between threads that could cause high CPU usage.
- \#3931 Fixed error caused by AMP running in clustered installations.
- \#3966 Changed type of msg & body columns in muc_history table for SQLServer to prevent loss of special characters.
- \#3973 Adjusted throttling settings for S2S and cluster connections.
- \#3977 Fixed MUC History to reflect messages from JID of room and not JID of original sender.
- \#3984 Fixed distinct usage on large data which causes errors on lookup of PubSub nodes in MongoDB.
- [#3970](https://projects.tigase.org/issues/3970) Fixed duplication of messages with AMP payload in cluster mode.
- [#4044](https://projects.tigase.org/issues/4044) Fixed various web installer issues.
- \#4051 Fixed NPE in java when processing message with no body.
- [#4052](https://projects.tigase.org/issues/4052) Fixed issue with ClusterRepoItem not properly resulting in tigase.db.comp.RepositoryChangeListenerIfc.itemUpdated(Item) being executed.
- [#4056](https://projects.tigase.org/issues/4056) Items removed from cluster repository are not removed from memory correctly.
- [#4071](https://projects.tigase.org/issues/4071) Updated groovy script to properly add owner to node creation VIA ad-hoc command.
- [#4142](https://projects.tigase.org/issues/4142) Updated wrapper.conf file to match tigase.conf default settings.
- [#4183](https://projects.tigase.org/issues/4183) Fixed issue where objects monitored by Ghostbuster.java in MUC could not be removed by it.
- [#4185](https://projects.tigase.org/issues/4185) Fixed issue with PacketCounter that caused duplicate messages to be sent on stream resumption.
- [#4188](https://projects.tigase.org/issues/4188) Standardized timestamp between AbstractMessageArchiveRepository and TimestampHelper.
- [#4262](https://projects.tigase.org/issues/4262) Fixed messages getting lost when StreamResumption is used when a disconnected user reconnects to the server. This issue is also fixed on servers using ACS component.
- [#4298](https://projects.tigase.org/issues/4298) ACS - Fixed messages getting dropped when sent to users offline or on unstable connections.
- [#4365](https://projects.tigase.org/issues/4365) Fixed direct presence not working with non-roster elements using barejid.
- \#4672 Fixed `UnsupportedOperationException` error occuring during configuration change of `WebSocketClientConnectionClustered`.
- Patch added to fix ConcurrentModificationException in BlockingCommand plugin.
- Fixed negation in SASL mechanism selector.
- Fixed checking for user session without localpart in to address.
- Fixed resourceDefPrefix from accumulating resource names when components are added or removed in web console.
- Distributed EventBus improved to allow POJO based events to be fired locally.
- Added missing classes to IzPack installer.
- Tigase.xml removed from documentation and default tigase.conf file.
- Logs function added to eventbus publisher operations.
- Fixed responding to same hostname as sender as "to" in stream-error stanza.
- Fixed issue where attempts to delete empty MUC room would create and then destroy room.
- Added startup information to log to indicate when server is ready.
- Fixed a divide by zero error in Java Garbage Collection.
- Fixed -see-other-host causing 'Foreign key constraint is incorrectly formed' error while using SeeOtherHostDB.



