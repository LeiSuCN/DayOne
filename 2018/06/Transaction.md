## [1.事务的概念](https://en.wikipedia.org/wiki/Database_transaction)
### 1.1 ACID
A:atomic, C:consistent, I:isolated, D:durable

### [1.2 事务的隔离级别](https://en.wikipedia.org/wiki/Isolation_(database_systems))
#### 1.2.1 读现象(Read phenomena)
- 脏读(dirty read): when a transaction is allowed to read data from a row that has been modified by another running transaction and not yet committed.
- 不可重复读(non-repeatable read): when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.
- 幻读(phantom read ): when, in the course of a transaction, new rows are added by another transaction to the records being read.
#### 1.2.2 隔离级别
- 串行(Serializable)
- 可重复读(Repeatable read)
- 读已提交(Read committed)
- 读未提交(Read uncommitted)

### 1.3数据库默认隔离级别
- mysql：Repeatable read
- oracle：Read committed
- sql server：Read committed

## 2.各组件中的事务管理
### 2.1 Spring
### 2.2 Mybatis

## 3.分布式事务
