# Tigase开发者指南

## 测试

### 测试

在开发过程中，测试是一个整个过程中的重要部分。

所有的release版本都是经过全自动化测试的。所有服务器的功能被实现都是通过测试后才会发布的。Tigase测试套件用于所有我们的自动测试，允许定义不同的测试场景。

我们对数据库测试没有进行调整。所有的数据库都以标准方式安装，并以默认方式运行。在每次循环测试之前，数据库数据都被清理。

同时，我们没有对默认配置进行更改。所有测试都是在配置向导生成的默认配置下执行的。

服务器在所有受支持的环境中进行测试：

- **XMLDB**,通过简单的XML database进行测试，对于小型服务安装，这是最简单和有效的方式。我们推荐它为100个用户的服务。虽然，我们对此通过了10000人的测试。

- **MySQL** ，使用mysql进行测试，虽然mysql比XMLDB慢得多，但是适用于更多的用户使用。

- **PostgreSQL** ，使用PostgreSQL数据库进行测试。再次，它比XMLDB慢得多，但可以处理更多的用户帐户。这与mysql的代码基本相同，但执行测试以确保他可以在支持sql的数据库上成功运行和性能比较。

- **Distributed** ，为分布式安装的测试。其中c2s和s2s组件在分离的机器上运行，该机器使用外部组件协议（XEP-0114）与运行SessionManager的另一台机器连接。

  ### 功能测试

  检查所有的功能运行是否正确。当代码被推送到源代码仓库上时，记性自动化测试。

  | Version    | XMLDB                                    | MySQL                                    | PGSQL                                    | Distributed                              |
  | ---------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
  | 3.3.2-b889 | [00:00:12](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/func/xmldb/functional-tests.html) | [00:00:17](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/func/mysql/functional-tests.html) | [00:00:17](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/func/pgsql/functional-tests.html) | none                                     |
  | 3.3.2-b880 | [00:00:13](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/func/xmldb/functional-tests.html) | [00:00:15](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/func/mysql/functional-tests.html) | [00:00:15](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/func/pgsql/functional-tests.html) | None                                     |
  | 3.0.2-b700 | [00:00:22](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/func/xmldb/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/func/mysql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/func/pgsql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/func/sm-mysql/functional-tests.html) |
  | 2.9.5-b606 | [00:00:22](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/func/xmldb/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/func/mysql/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/func/pgsql/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/func/sm-mysql/functional-tests.html) |
  | 2.9.3-b548 | [00:00:22](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/func/xmldb/functional-tests.html) | [00:00:23](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/func/mysql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/func/pgsql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/func/sm-mysql/functional-tests.html) |
  | 2.9.1-b528 | [00:00:21](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/func/xmldb/functional-tests.html) | [00:00:23](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/func/mysql/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/func/pgsql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/func/sm-mysql/functional-tests.html) |
  | 2.8.6-b434 | [00:00:21](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/func/xmldb/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/func/mysql/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/func/pgsql/functional-tests.html) | [00:00:25](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/func/sm-mysql/functional-tests.html) |
  | 2.8.5-b422 | [00:00:21](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/func/xmldb/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/func/mysql/functional-tests.html) | [00:00:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/func/pgsql/functional-tests.html) | [00:00:26](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/func/sm-mysql/functional-tests.html) |
  | 2.8.3-b409 | [00:00:27](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/func/xmldb/functional-tests.html) | [00:00:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/func/mysql/functional-tests.html) | [00:00:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/func/pgsql/functional-tests.html) | [00:00:32](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/func/sm-mysql/functional-tests.html) |
  | 2.7.2-b378 | [00:00:30](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/func/xmldb/functional-tests.html) | [00:00:34](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/func/mysql/functional-tests.html) | [00:00:33](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/func/pgsql/functional-tests.html) | [00:00:35](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/func/sm-mysql/functional-tests.html) |
  | 2.6.4-b300 | [00:00:30](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/func/xmldb/functional-tests.html) | [00:00:32](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/func/mysql/functional-tests.html) | [00:00:35](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/func/pgsql/functional-tests.html) | [00:00:39](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/func/sm-mysql/functional-tests.html) |
  | 2.6.4-b295 | [00:00:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/func/xmldb/functional-tests.html) | [00:00:32](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/func/mysql/functional-tests.html) | [00:00:45](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/func/pgsql/functional-tests.html) | [00:00:36](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/func/sm-mysql/functional-tests.html) |
  | 2.6.0-b287 | [00:00:31](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/func/xmldb/functional-tests.html) | [00:00:34](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/func/mysql/functional-tests.html) | [00:00:47](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/func/pgsql/functional-tests.html) | [00:00:43](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/func/sm-mysql/functional-tests.html) |
  | 2.5.0-b279 | [00:00:30](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/func/xmldb/functional-tests.html) | [00:00:34](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/func/mysql/functional-tests.html) | [00:00:45](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/func/pgsql/functional-tests.html) | [00:00:43](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/func/sm-mysql/functional-tests.html) |
  | 2.4.0-b263 | [00:00:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/func/xmldb/functional-tests.html) | [00:00:33](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/func/mysql/functional-tests.html) | [00:00:45](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/func/pgsql/functional-tests.html) | [00:00:44](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/func/sm-mysql/functional-tests.html) |
  | 2.3.4-b226 | None                                     | [00:00:48](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/functional-tests.html) | None                                     | None                                     |

  ### 性能测试

  检查所有的功能是否运行良好

  | Version    | XMLDB                                    | MySQL                                    | PGSQL                                    | Distributed                              |
  | ---------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
  | 3.3.2-b889 | [00:12:17](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/perf/xmldb/performance-tests.html) | [00:13:42](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/perf/mysql/performance-tests.html) | [00:17:10](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b889/perf/pgsql/performance-tests.html) | none                                     |
  | 3.3.2-b880 | [00:13:39](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/perf/xmldb/performance-tests.html) | [00:14:09](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/perf/mysql/performance-tests.html) | [00:17:39](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.3.2-b880/perf/pgsql/performance-tests.html) | None                                     |
  | 3.0.2-b700 | [00:10:26](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/perf/xmldb/performance-tests.html) | [00:11:00](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/perf/mysql/performance-tests.html) | [00:12:08](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/perf/pgsql/performance-tests.html) | [00:24:05](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/3.0.2-b700/perf/sm-mysql/performance-tests.html) |
  | 2.9.5-b606 | [00:09:54](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/perf/xmldb/performance-tests.html) | [00:11:18](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/perf/mysql/performance-tests.html) | [00:37:08](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/perf/pgsql/performance-tests.html) | [00:16:20](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.5-b606/perf/sm-mysql/performance-tests.html) |
  | 2.9.3-b548 | [00:10:00](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/perf/xmldb/performance-tests.html) | [00:11:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/perf/mysql/performance-tests.html) | [00:36:43](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/perf/pgsql/performance-tests.html) | [00:16:47](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.3-b548/perf/sm-mysql/performance-tests.html) |
  | 2.9.1-b528 | [00:09:46](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/perf/xmldb/performance-tests.html) | [00:11:15](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/perf/mysql/performance-tests.html) | [00:36:12](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/perf/pgsql/performance-tests.html) | [00:16:36](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.9.1-b528/perf/sm-mysql/performance-tests.html) |
  | 2.8.6-b434 | [00:10:02](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/perf/xmldb/performance-tests.html) | [00:11:45](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/perf/mysql/performance-tests.html) | [00:36:36](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/perf/pgsql/performance-tests.html) | [00:17:36](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.6-b434/perf/sm-mysql/performance-tests.html) |
  | 2.8.5-b422 | [00:12:37](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/perf/xmldb/performance-tests.html) | [00:14:40](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/perf/mysql/performance-tests.html) | [00:38:59](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/perf/pgsql/performance-tests.html) | [00:21:40](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.5-b422/perf/sm-mysql/performance-tests.html) |
  | 2.8.3-b409 | [00:12:32](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/perf/xmldb/performance-tests.html) | [00:14:26](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/perf/mysql/performance-tests.html) | [00:37:57](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/perf/pgsql/performance-tests.html) | [00:21:26](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.8.3-b409/perf/sm-mysql/performance-tests.html) |
  | 2.7.2-b378 | [00:12:28](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/perf/xmldb/performance-tests.html) | [00:14:57](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/perf/mysql/performance-tests.html) | [00:37:09](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/perf/pgsql/performance-tests.html) | [00:22:20](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.7.2-b378/perf/sm-mysql/performance-tests.html) |
  | 2.6.4-b300 | [00:12:46](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/perf/xmldb/performance-tests.html) | [00:14:59](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/perf/mysql/performance-tests.html) | [00:36:56](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/perf/pgsql/performance-tests.html) | [00:35:00](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b300/perf/sm-mysql/performance-tests.html) |
  | 2.6.4-b295 | [00:12:23](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/perf/xmldb/performance-tests.html) | [00:14:59](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/perf/mysql/performance-tests.html) | [00:42:24](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/perf/pgsql/performance-tests.html) | [00:30:18](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.4-b295/perf/sm-mysql/performance-tests.html) |
  | 2.6.0-b287 | [00:13:50](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/perf/xmldb/performance-tests.html) | [00:16:53](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/perf/mysql/performance-tests.html) | [00:48:17](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/perf/pgsql/performance-tests.html) | [00:49:06](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.6.0-b287/perf/sm-mysql/performance-tests.html) |
  | 2.5.0-b279 | [00:13:29](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/perf/xmldb/performance-tests.html) | [00:16:58](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/perf/mysql/performance-tests.html) | [00:47:15](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/perf/pgsql/performance-tests.html) | [00:41:52](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.5.0-b279/perf/sm-mysql/performance-tests.html) |
  | 2.4.0-b263 | [00:13:20](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/perf/xmldb/performance-tests.html) | [00:16:21](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/perf/mysql/performance-tests.html) | [00:43:56](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/perf/pgsql/performance-tests.html) | [00:42:08](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/2.4.0-b263/perf/sm-mysql/performance-tests.html) |
  | 2.3.4-b226 | None                                     | [01:23:30](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/performance-tests.html) | None                                     | None                                     |

  ### 稳定性测试

  检查长时间运行是否运行良好。他必须在几小时内处理成千上万的请求。

  | Version    | XMLDB | MySQL                                    | PGSQL | Distributed |
  | ---------- | ----- | ---------------------------------------- | ----- | ----------- |
  | 2.3.4-b226 | None  | [16:06:31](http://docs.tigase.org/tigase-server/7.0.1/Development_Guide/html/tests/stability-tests.html) | None  | None        |

  ## Tigase DB模式解释

  对于最基本的模式，对于基本的外观和简明概要可以在[schema creation script](https://projects.tigase.org/projects/tigase-server/repository/revisions/master/entry/database/mysql-schema-4-schema.sql)中查看。然而，看到上面的脚本，我们很难理解他是如何工作的。是怎样访问数据的。其实，在所有的表中，只有三个基本的表保存了用户的数据。分别是`tig_users`,`tig_nodes`,`tig_pairs`。但我们还是不清楚Tigase是如何被组织的。

  在你可以理解Tigase XMPP Server数据库模式，如何工作，如何使用之前。理解它为什么这样运行，并且知道这样运行的目标是什么。让我们从API开始，因为这将给你最好的介绍。

  可以通过以下的方法来进行简单的访问：

  ```java
  void setData(BareJID user, String key, String value);
  String getData(BareJID user, String key);
  ```

  更多复杂的版本：

  ```java
  void setData(BareJID user, String subnode, String key, String value);
  String getData(BareJID user, String subnode, String key, String def);
  ```

