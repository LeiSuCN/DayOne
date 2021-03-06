## [1.事务的概念](https://en.wikipedia.org/wiki/Database_transaction)
### 1.1 ACID
A - atomic, C - consistent, I - isolated, D - durable

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
#### 2.1.1 核心接口
```java
public interface PlatformTransactionManager {

	TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

	void commit(TransactionStatus status) throws TransactionException;

	void rollback(TransactionStatus status) throws TransactionException;

}
```

#### 2.1.2 其他接口
```java
public interface TransactionDefinition {

	int PROPAGATION_REQUIRED = 0;
	int PROPAGATION_SUPPORTS = 1;
	int PROPAGATION_MANDATORY = 2;
	int PROPAGATION_REQUIRES_NEW = 3;
	int PROPAGATION_NOT_SUPPORTED = 4;
	int PROPAGATION_NEVER = 5;
	int PROPAGATION_NESTED = 6;
	int ISOLATION_DEFAULT = -1;
	int ISOLATION_READ_UNCOMMITTED = Connection.TRANSACTION_READ_UNCOMMITTED;
	int ISOLATION_READ_COMMITTED = Connection.TRANSACTION_READ_COMMITTED;
	int ISOLATION_REPEATABLE_READ = Connection.TRANSACTION_REPEATABLE_READ;
	int ISOLATION_SERIALIZABLE = Connection.TRANSACTION_SERIALIZABLE;

	int getPropagationBehavior();

	int getIsolationLevel();

	int getTimeout();

	boolean isReadOnly();

	String getName();

}
```
```java
public interface TransactionStatus extends SavepointManager, Flushable {

	boolean isNewTransaction();

	boolean hasSavepoint();

	void setRollbackOnly();

	boolean isRollbackOnly();

	@Override
	void flush();

	boolean isCompleted();

}
```
```java
public interface SavepointManager {

	Object createSavepoint() throws TransactionException;

	void rollbackToSavepoint(Object savepoint) throws TransactionException;

	void releaseSavepoint(Object savepoint) throws TransactionException;

}
```

### 2.2 Mybatis
#### 2.2.1 核心接口
```java
public interface Transaction {

  Connection getConnection() throws SQLException;

  void commit() throws SQLException;

  void rollback() throws SQLException;

  void close() throws SQLException;

  Integer getTimeout() throws SQLException;
  
}
```

## 3.分布式事务
