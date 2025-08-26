+++
date = '2025-08-26T20:41:36+08:00'
draft = false
title = '数据库 - 总结'
categories = ['Sub Sections']
tags = ['数据库']
+++

## 目前主流的数据库
目前主流的数据库有： Oracle db, MySQL, SQL Server, PostgreSQL 。以及还有一个较为特殊的数据库： SQLite 。

### Oracle db
Oracle db 是甲骨文公司的，完全商用的数据库。它不能云部署，只能私有部署。同时由于这个数据库是收费的，使用的人不多。

### MySQL
一个字：菜！别用。

参考链接：

| 标题 | Bilibili | YouTube |
| :--: | :--: | :--: |
| 千疮百孔的世界上最流行的数据库 | https://www.bilibili.com/video/BV1BAABeKEqp | https://www.youtube.com/watch?v=KpJCHsRcZ1g |
| 有些数据库是蠢，有些是坏，它是又蠢又坏 | https://www.bilibili.com/video/BV1M1Koz6EW1 | https://www.youtube.com/watch?v=ConMAwL-cmk |
| 全网最尊重MySQL的博主 | https://www.bilibili.com/video/BV12We7z8EZR | https://www.youtube.com/watch?v=UozK9-IwOBE |

### SQL Server
SQL Server 是由微软开发的数据库。以前是收费的。现在有 Express 版本，可用于小型系统。

### PostgreSQL
PostgreSQL 是开源免费的，使用 MIT 开源协议。 PostgreSQL 的功能十分丰富和强大，也因此，在实际使用中，需要由专门的 DBA 负责管理。

参考链接：

| 标题 | Bilibili | YouTube |
| :--: | :--: | :--: |
| PostgreSQL能存万物！这还是你认识的数据库吗？ | https://www.bilibili.com/video/BV1FUYQz7E4H | https://www.youtube.com/watch?v=1UPoCK0v22w |

### SQLite
SQLite 的特殊之处在于，它没有独立的数据库管理系统。诸如 PostgreSQL 、 SQL Server 等数据库，在使用前，需要先在系统中安装对应的数据库管理系统。 SQLite 不需要这样，它的数据库管理系统可以集成在你开发的应用程序中，各大语言都有对应的软件包，直接在代码中使用即可。

SQLite 的数据库文件是单个文件，后缀一般为 `.db` `.sqlite` 。不过也并不总是单个文件，有时它还会生成 `.db-shm` 以及 `.db-wal` 。

> `.db-shm` 是共享内存文件，用于控制并发。

> `.db-wal` 是写前日志文件。

由于没有独立的数据库管理系统，数据库文件还是单个文件，所以 SQLite 的并发能力极弱，或者说，它几乎没有并发能力，因为数据库文件是单个文件，同一时间只能由一个进程访问。

SQLite 常用于低性能的，或者不方便安装数据库管理系统的环境，比如安卓手机、苹果手机、嵌入式设备等。

除此以外， SQLite 还能作为内存数据库使用。

## See Also
| 标题 | Bilibili | YouTube |
| :--: | :--: | :--: |
| 我为什么把业务逻辑都写进数据库 | https://www.bilibili.com/video/BV1si421a7aE | https://www.youtube.com/watch?v=BWI943IEHIk |
| 为什么关系数据库的挑战者都没有好下场 | https://www.bilibili.com/video/BV1i9rfYKEw7 | https://www.youtube.com/watch?v=IUUpxfa1SSw |
