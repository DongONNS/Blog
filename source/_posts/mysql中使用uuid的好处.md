---
title: MySQL中自增主键的好处
date: 2021-01-26 22:00:00
tags:
categories: MySQL
---

# UUID 好处 以及 自增主键 的优缺点

自增主键
自增ID是在设计表时将id字段的值设置为自增的形式，这样当插入一行数据时无需指定id会自动根据前一字段的ID值+1进行填充。在[MySQL](https://www.yisu.com/mysql/)数据库中，可通过sql语句AUTO_INCREMENT来对特定的字段启用自增赋值 使用自增ID作为主键，能够保证字段的原子性.

优点

数据库自动编号，速度快，而且是增量增长，按顺序存放，对于检索非常有利；
数字型，占用空间小，易排序，在程序中传递也方便；
如果通过非系统增加记录时，可以不用指定该字段，不用担心主键重复问题。
缺点

因为自动增长，在手动要插入指定ID的记录时会显得麻烦，尤其是当系统与其它系统集成时，需要数据导入时，很难保证原系统的ID不发生主键冲突（前提是老系统也是数字型的）。特别是在新系统上线时，新旧系统并行存在，并且是异库异构的数据库的情况下，需要双向同步时，自增主键将是你的噩梦；
在系统集成或割接时，如果新旧系统主键不同是数字型就会导致修改主键数据类型，这也会导致其它有外键关联的表的修改，后果同样很严重；
若系统也是数字型的，在导入时，为了区分新老数据，可能想在老数据主键前统一加一个字符标识（例如“o”，old）来表示这是老数据，那么自动增长的数字型又面临一个挑战。
UUID
UUID含义是通用唯一识别码 (Universally Unique Identifier)uuid 项目应用 www.1b23.com，指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。通常平台会提供生成的API。换句话说能够在一定的范围内保证主键id的唯一性。

优点

出现数据拆分、合并存储的时候，能达到全局的唯一性

缺点

影响插入速度， 并且造成硬盘使用率低
uuid之间比较大小相对数字慢不少， 影响查询速度。
uuid占空间大， 如果你建的索引越多， 影响越严重