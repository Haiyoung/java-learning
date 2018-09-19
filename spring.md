<!-- TOC -->

- [Spring](#spring)
    - [spring事物](#spring%E4%BA%8B%E7%89%A9)
        - [spring事物特性](#spring%E4%BA%8B%E7%89%A9%E7%89%B9%E6%80%A7)
        - [spring事物的传播机制](#spring%E4%BA%8B%E7%89%A9%E7%9A%84%E4%BC%A0%E6%92%AD%E6%9C%BA%E5%88%B6)
        - [spring事物的隔离级别](#spring%E4%BA%8B%E7%89%A9%E7%9A%84%E9%9A%94%E7%A6%BB%E7%BA%A7%E5%88%AB)
        - [脏读 幻读 不可重复读](#%E8%84%8F%E8%AF%BB-%E5%B9%BB%E8%AF%BB-%E4%B8%8D%E5%8F%AF%E9%87%8D%E5%A4%8D%E8%AF%BB)
        - [reference](#reference)
    - [spring bean的生命周期和作用域](#spring-bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E4%BD%9C%E7%94%A8%E5%9F%9F)

<!-- /TOC -->
## Spring
### spring事物
#### spring事物特性
- Atomicity 原子性,一个事务中所有对数据库的操作是一个不可分割的操作序列，要么全做要么全不做
- Consistency 一致性,数据不会因为事务的执行而遭到破坏,事务必须始终保持系统处于一致的状态
- Isolation 隔离性,一个事物的执行，不受其他事务的干扰，即并发执行的事物之间互不干扰
- Durability 持久性,一个事物一旦提交，它对数据库的改变就是永久的

#### spring事物的传播机制
spring事务的传播机制，就是定义在存在多个事务同时存在的时候，spring应该如何处理这些事务的行为
- propagation_required 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是spring默认的事务的传播
- propagation_requires_new 新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作 
- propagation_supports 支持当前事务，如果当前没有事务，就以非事务方式执行
- propagation_mandatory 支持当前事务，如果当前没有事务，就抛出异常
- propagation_not_supported 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
- propagation_never 以非事务方式执行，如果当前存在事务，则抛出异常
- propagation_nested 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效

#### spring事物的隔离级别
- isolation_default PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别
- isolation_read_uncommitted 事物最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。 这种隔离级别会产生脏读，不可重复读和幻读
- isolation_read_committed 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。 这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读
- isolation_repeatable_read 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻读。 
- isolation_serializable 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行，除了防止脏读，不可重复读外，还避免了幻读

#### 脏读 幻读 不可重复读
脏读：指一个事务读取了一个未提交事务的数据
不可重复读：在一个事务内读取表中的某一行数据,多次读取结果不同.一个事务读取到了另一个事务提交后的数据.
幻读：在一个事务内读取了别的事务插入的数据，导致前后读取不一致
#### reference
- [https://www.jdon.com/concurrent/acid-database.html](https://www.jdon.com/concurrent/acid-database.html)

### spring bean的生命周期和作用域
