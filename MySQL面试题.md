# 📚 MySQL 面试题大全

## 🏗️ 数据表设计

### 1. 什么是数据库范式？第三范式有什么特点？
**答案**：
数据库范式是设计数据库时遵循的规范，第三范式(3NF)要求：
1. 满足第二范式
2. 消除非主属性对码的传递依赖
3. 每个非主属性必须直接依赖于主键

### 2. 如何设计一个用户-角色-权限系统？
**答案**：
典型设计包含5张表：
1. `users` - 用户基本信息
2. `roles` - 角色定义
3. `permissions` - 权限定义
4. `user_roles` - 用户角色关联
5. `role_permissions` - 角色权限关联

## 🔍 SQL查询

### 3. 如何查询每个部门薪资最高的员工？
**答案**：
```sql
SELECT e.* FROM employees e
JOIN (
    SELECT department_id, MAX(salary) max_salary
    FROM employees
    GROUP BY department_id
) d ON e.department_id = d.department_id AND e.salary = d.max_salary;

```
---
### 4. 如何优化以下慢查询？
```sql
SELECT * FROM orders WHERE DATE(create_time) = '2023-01-01';
```
**答案**：
改为范围查询：
```sql
SELECT * FROM orders 
WHERE create_time >= '2023-01-01 00:00:00' 
AND create_time < '2023-01-02 00:00:00';
```

## 🚀 索引优化

### 5. 什么情况下索引会失效？
**答案**：
1. 使用`!=`、`<>`、`NOT IN`等否定条件
2. 对索引列使用函数或计算
3. 隐式类型转换
4. 使用前导模糊查询`LIKE '%xxx'`
5. 复合索引不满足最左前缀原则

### 6. 如何判断一个SQL是否使用了索引？
**答案**：
使用`EXPLAIN`命令查看执行计划，关注：
1. `type`列：`index`/`range`表示使用索引
2. `key`列：显示使用的索引名称
3. `rows`列：扫描行数

## 💼 事务管理

### 7. 事务的ACID特性是什么？
**答案**：
1. **A**tomicity(原子性)：事务是不可分割的工作单位
2. **C**onsistency(一致性)：事务执行前后数据库保持一致状态
3. **I**solation(隔离性)：并发事务间互不干扰
4. **D**urability(持久性)：事务提交后永久生效

### 8. MySQL有哪些隔离级别？默认是什么？
**答案**：
4种隔离级别：
1. 读未提交(READ UNCOMMITTED)
2. 读已提交(READ COMMITTED)
3. 可重复读(REPEATABLE READ) - **默认级别**
4. 串行化(SERIALIZABLE)

## 🔒 锁机制

### 9. InnoDB有哪几种行锁？
**答案**：
1. 记录锁(Record Lock)：锁定索引记录
2. 间隙锁(Gap Lock)：锁定索引记录间隙
3. 临键锁(Next-Key Lock)：记录锁+间隙锁组合
4. 插入意向锁(Insert Intention Lock)

### 10. 什么是死锁？如何避免？
**答案**：
死锁是多个事务互相等待对方释放锁的情况。避免方法：
1. 按固定顺序访问表和行
2. 减小事务范围
3. 设置合理的锁等待超时时间
4. 使用`SHOW ENGINE INNODB STATUS`分析死锁

## 🏎️ 存储引擎

### 11. InnoDB和MyISAM的主要区别？
**答案**：
| 特性        | InnoDB               | MyISAM          |
|------------|---------------------|----------------|
| 事务支持    | 支持                 | 不支持          |
| 外键        | 支持                 | 不支持          |
| 锁粒度      | 行锁                 | 表锁            |
| 崩溃恢复    | 支持                 | 不支持          |
| 全文索引    | MySQL5.6+支持        | 支持            |

### 12. 什么场景适合使用Memory引擎？
**答案**：
适合场景：
1. 临时表/中间结果集
2. 只读或极少更新的数据
3. 需要极快访问速度
4. 数据量小且可接受丢失

## 🚦 高并发处理

### 13. 如何解决超卖问题？
**答案**：
1. 乐观锁：版本号控制
2. 悲观锁：`SELECT ... FOR UPDATE`
3. Redis分布式锁
4. 消息队列削峰

### 14. 大表分页查询如何优化？
**答案**：
1. 使用延迟关联：
```sql
SELECT * FROM orders o
JOIN (SELECT id FROM orders ORDER BY id LIMIT 100000, 10) t ON o.id = t.id;
```
2. 记录上次查询的最大ID
3. 使用覆盖索引

## ⚙️ 性能调优

### 15. 如何分析慢查询？
**答案**：
1. 开启慢查询日志
2. 使用`mysqldumpslow`分析日志
3. `EXPLAIN`分析执行计划
4. 使用`pt-query-digest`工具

### 16. 如何优化JOIN查询？
**答案**：
1. 确保关联字段有索引
2. 小表驱动大表
3. 避免`SELECT *`，只查询必要字段
4. 考虑使用子查询替代JOIN

## 🧩 高级特性

### 17. 什么是覆盖索引？有什么优点？
**答案**：
覆盖索引是指查询的列都包含在索引中。优点：
1. 避免回表操作
2. 减少IO开销
3. 提升查询性能

### 18. 什么是MRR优化？
**答案**：
Multi-Range Read优化，工作流程：
1. 先扫描索引收集主键
2. 对主键排序
3. 按主键顺序访问数据
减少随机IO，提升范围查询性能

## 🛠️ 运维相关

### 19. 如何安全地修改大表结构？
**答案**：
1. 使用`pt-online-schema-change`工具
2. 低峰期操作
3. 先备份再修改
4. 考虑创建新表后数据迁移

### 20. 如何监控MySQL性能？
**答案**：
常用监控指标：
1. QPS/TPS
2. 连接数
3. 缓存命中率
4. 慢查询数量
工具：
1. `SHOW STATUS`
2. `SHOW PROCESSLIST`
3. Performance Schema
4. Prometheus+Granfana

## 🔄 主从复制

### 21. MySQL主从复制原理是什么？
**答案**：
基于binlog的异步复制流程：
1. 主库记录所有DDL/DML到binlog
2. 从库I/O线程请求主库的binlog
3. 主库dump线程发送binlog给从库
4. 从库SQL线程重放binlog中的SQL

### 22. 主从延迟有哪些原因？如何解决？
**答案**：
原因：
1. 主库并发事务量大
2. 从库硬件性能差
3. 大事务执行时间长
解决方案：
1. 使用GTID复制
2. 开启并行复制
3. 优化大事务

## 🧠 查询优化

### 61. 如何优化大表COUNT(*)查询？
**答案**：
1. 使用近似值：`EXPLAIN SELECT COUNT(*)`
2. 维护计数表
3. 使用`information_schema.TABLES`估算
4. 对MyISAM表直接查询元数据

### 62. 什么是索引下推？有什么优势？
**答案**：
索引下推(ICP)将WHERE条件过滤下推到存储引擎层执行，减少回表操作，提升查询性能5-10倍。

## 🛡️ 数据安全

### 63. 如何防止SQL注入？
**答案**：
1. 使用预编译语句
2. 参数化查询
3. 输入验证和过滤
4. 最小权限原则

### 64. MySQL有哪些备份方法？
**答案**：
1. 逻辑备份：mysqldump
2. 物理备份：Percona XtraBackup
3. 二进制日志备份
4. 快照备份

## ⚡ 性能调优

### 65. 连接池大小如何设置？
**答案**：
公式：连接数 = (核心数 * 2) + 有效磁盘数。监控`Threads_connected`和`Threads_running`调整。

### 66. 如何优化大表JOIN？
**答案**：
1. 确保关联字段有索引
2. 使用小表驱动大表
3. 考虑分批次JOIN
4. 使用临时表

### 41. 如何优化GROUP BY查询？
**答案**：
1. 确保GROUP BY字段有索引
2. 使用ORDER BY NULL避免排序
3. 考虑使用临时表
4. 适当增加tmp_table_size

### 42. 如何优化子查询？
**答案**：
1. 使用JOIN替代子查询
2. 使用EXISTS替代IN
3. 将子查询改为派生表
4. 使用索引优化子查询

## 🛡️ 安全机制

### 43. MySQL有哪些权限控制级别？
**答案**：
1. 全局权限：*.*
2. 数据库权限：db_name.*
3. 表权限：db_name.table_name
4. 列权限
5. 存储过程和函数权限

### 44. 如何审计MySQL操作？
**答案**：
1. 开启general_log
2. 使用审计插件如audit_log
3. 使用binlog分析工具
4. 第三方审计工具

## 🚀 性能分析

### 45. 如何分析MySQL内存使用？
**答案**：
1. `SHOW ENGINE INNODB STATUS`
2. `SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%'`
3. `performance_schema`库
4. 使用pt-mysql-summary工具

### 46. 如何定位CPU高负载问题？
**答案**：
1. `SHOW PROCESSLIST`查看运行线程
2. `performance_schema`分析SQL
3. 检查慢查询日志
4. 使用pt-query-digest

## 🔄 分布式

### 47. 什么是分库分表？有哪些策略？
**答案**：
分库分表是将数据分散到不同数据库/表的策略：
1. 水平分表：按行拆分
2. 垂直分表：按列拆分
3. 水平分库：不同库存储不同数据
4. 垂直分库：按业务拆分

### 48. 分库分表有哪些挑战？
**答案**：
1. 分布式事务
2. 跨库JOIN
3. 全局唯一ID
4. 分页查询
5. 数据迁移

## 💾 存储优化

### 49. InnoDB缓冲池如何调优？
**答案**：
1. 设置合理的innodb_buffer_pool_size
2. 监控命中率(>99%)
3. 预热缓冲池
4. 使用多缓冲池实例

### 50. 如何优化磁盘IO？
**答案**：
1. 使用SSD
2. 调整innodb_io_capacity
3. 分离数据和日志文件
4. 使用RAID

## 🧩 高级特性

### 51. 什么是窗口函数？有哪些？
**答案**：
窗口函数对查询结果的子集进行计算：
1. ROW_NUMBER()
2. RANK()
3. DENSE_RANK()
4. LEAD()/LAG()
5. FIRST_VALUE()/LAST_VALUE()

### 52. 什么是CTE？有什么优点？
**答案**：
公共表表达式(WITH子句)优点：
1. 提高复杂查询可读性
2. 支持递归查询
3. 可多次引用
4. 替代临时表

## 🛠️ 运维实战

### 53. 如何在线修改大表结构？
**答案**：
1. 使用pt-online-schema-change
2. 使用gh-ost
3. MySQL8.0+的instant ADD COLUMN
4. 低峰期操作

### 54. 如何快速重建大表索引？
**答案**：
1. 使用ALTER TABLE ... ALGORITHM=INPLACE
2. 分批次操作
3. 低峰期执行
4. 考虑使用pt-index-usage分析索引使用

## 🔍 诊断工具

### 55. 如何分析锁等待？
**答案**：
1. `SHOW ENGINE INNODB STATUS`
2. `performance_schema.events_waits_current`
3. `sys.innodb_lock_waits`
4. `information_schema.INNODB_TRX`

### 56. 如何诊断性能瓶颈？
**答案**：
1. 使用`EXPLAIN ANALYZE`
2. 分析`performance_schema`
3. 使用pt-pmp
4. 检查操作系统指标

## 🚦 高可用架构

### 57. 什么是MGR？有什么特点？
**答案**：
MySQL Group Replication特点：
1. 基于Paxos协议
2. 多主/单主模式
3. 自动故障检测
4. 数据一致性保证

### 58. 如何设计读写分离架构？
**答案**：
1. 使用ProxySQL中间件
2. 基于GTID的复制
3. 读写分离路由策略
4. 故障自动转移

## 💡 新特性

### 59. MySQL8.0有哪些重要新特性？
**答案**：
1. 窗口函数
2. 通用表表达式(CTE)
3. 原子DDL
4. 不可见索引
5. 资源组

### 60. 什么是JSON数据类型？如何使用？
**答案**：
JSON类型提供：
1. JSON_VALID()验证
2. ->/->>操作符
3. JSON_SET()等修改函数
4. 支持索引(生成列)

## 🔄 分布式事务

### 67. 什么是XA事务？如何实现？
**答案**：
XA是分布式事务标准，通过两阶段提交实现：
1. 准备阶段：协调者询问参与者是否可提交
2. 提交阶段：协调者通知参与者提交/回滚

### 68. MySQL如何支持分布式事务？
**答案**：
1. 使用XA协议
2. 通过`XA START/END/PREPARE/COMMIT`命令
3. 配合事务协调器(如Atomikos)

## 🧠 查询执行

### 69. 什么是执行计划？如何解读？
**答案**：
执行计划显示查询执行路径，关键列：
1. `type`：访问类型(ALL/index/range等)
2. `key`：使用的索引
3. `rows`：预估扫描行数
4. `Extra`：额外信息

### 70. 如何强制使用特定索引？
**答案**：
```sql
SELECT * FROM table USE INDEX(index_name) WHERE ...
-- 或
SELECT * FROM table FORCE INDEX(index_name) WHERE ...
```
---






        