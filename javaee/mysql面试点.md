---
typora-root-url: ..\img
---

## 第1章 MySQL架构介绍

### 1.1 MySQL逻辑架构

- 连接层 连接处理、授权认证
- 服务层 核心服务功能
- 引擎层 数据的存储和提取
- 存储层 文件系统

### <span style="color:red;">1.2 存储引擎(面试重点)</span>

MyISAM:不支持事务,不支持外键,表锁,只缓存索引,表空间小,关注点是性能

InnoDB:支持事务,支持主外键,行锁,缓存索引+数据,表空间大,关注点是事务

### 1.3 MySQL的影响因素

不适合存储在数据库的场景

多媒体数据 超大文本数据 流水数据

适合放在数据的信息

系统配置信息,用户信息,经常使用的数据

- OLTP 联机事务 事务的增删改查
- OLAP 分析事务  访问慢IO开销大

## 第2章 查询与索引优化分析

### 2.1 SQL执行顺序

![](/1568012410223.png)

### 2.2 join

![](/1568012124390.png)



## 第3章 索引与数据处理

### 1.1 索引

索引是排好序方便快速查找的数据结构

### 1.2 索引的优势与劣势

优势:提高数据的检索效率,降低排序成本

劣势:更新较慢

### 1.3 索引分类

单列索引,复合索引,唯一索引

### 1.4 性能优化

explain+SQL 可以

1. 查看表的读取顺序
2. 查询数据读物的操作类型
3. 哪些索引被使用,被实际使用
4. 表之间的引用
5. 每一张表有多少行被优化器查询

![](/1568014193766.png)

id 表示表的执行顺序 id相同顺序执行 id不同从大大小开始执行 

type表示使用了什么查询类型,要避免使用全表扫描 尽可能达到ref,range级别即扫描给定范围或者非唯一性扫描

key 表示实际使用的索引

rows 表示检索到给定数据需要读取的行数 越小越好

### 1.5查询优化

- 小表驱动大表
- Order By尽可能根据在索引列上,完成排序操作,按照索引创建的最佳左前缀法则
- Group By的实质是先排序后分组,所以按照索引列的最佳左前缀法则 

-------------------------

### 大数据量处理理论

### 主从复制

### MySQL锁机制

### 分库分区分表机制

