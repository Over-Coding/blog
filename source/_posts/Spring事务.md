---
title: Spring事务
date: 2020-01-07 15:09:27
categories:
	- Spring
tags: 
	- Spring
toc: true
---



### 1、简介
事务就是一组由于逻辑上紧密关联而合并成一个整体（工作单元）的多个数据库操作，这些操作要么都执行，要么都不执行。



### 2、事务的特性（ACID）

1. **原子性(atomicity)：**“原子”的本意是“不可再分”，事务的原子性表现为一个事务中涉及到的多个操作在逻辑上缺一不可。事务的原子性要求事务中的所有操作要么都执行，要么都不执行。
2. **一致性(consistency)：**“一致”指的是数据的一致，具体是指：所有数据都处于满足业务规则的一致性状态。一致性原则要求：一个事务中不管涉及到多少个操作，都必须保证事务执行之前数据是正确的，事务执行之后数据仍然是正确的。如果一个事务在执行的过程中，其中某一个或某几个操作失败了，则必须将其他所有操作撤销，将数据恢复到事务执行之前的状态，这就是回滚。
3. **隔离性(isolation)：** 在应用程序实际运行过程中，事务往往是并发执行的，所以很有可能有许多事务同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。隔离性原则要求多个事务在并发执行过程中不会互相干扰。
4. **持久性(durability)：** 持久性原则要求事务执行完成后，对数据的修改永久的保存下来，不会因各种系统错误或其他意外情况而受到影响。通常情况下，事务对数据的修改应该被写入到持久化存储器中。



### 3、Spring 事务管理器

Spring 从不同的事务管理API中抽象出了一整套事务管理机制，使开发人员可以通过配置的方式进行事务管理，而不必了解其底层是如何实现的。
Spring的核心事务管理抽象是 PlatformTransactionManager 接口：

```java
package org.springframework.transaction;

public interface PlatformTransactionManager { 

    TransactionStatus getTransaction(TransactionDefinition definition) throws                                                                                                     TransactionException; 
    
    void commit(TransactionStatus status) throws TransactionException; 
    
    void rollback(TransactionStatus status) throws TransactionException;
 }
```
PlatformTransactionManager 接口常用的实现类：

* DataSourceTransactionManager：应用于普通的JDBC。
* JtaTransactionManager：应用于JPA。
* HibernateTransactionManager：应用于Hibernate框架。
* TransactionTemplate：应用于编程式事务管理。



### 4、Spring 事务管理

Spring提供了两种事务管理的方式，分别是：

* 编程式事务管理

  编程式事务管理类似于使用原生的JDBC API实现事务管理，将事务管理的代码嵌入到业务代码中来控制事务的提交和回滚。但是事务管理的代码相同，会产生大量的代码冗余。

* 声明式事务管理（推荐使用）

  因编程式事务管理在代码中会产生大量的代码冗余，所以Spring基于Spring AOP框架实现了一种通过配置即可实现事务管理的方式，即声明式事务。只要方法中抛出RuntimeException或Error就会触发事务回滚。
  Spring提供了两种配置方式：

  * 注解@Transaction（推荐使用）
    @Transaction 可以放在方法上使用，也可以放在类上使用，如果放在类上使用就是为类中的所有方法以及子类中的所有方法都加上了事务。

  ```xml
  <!-- 配置数据源-->
  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"> 
      <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
      <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
      <property name="username" value="scott"/> 
      <property name="password" value="tiger"/> 
  </bean> 
  
  <!-- 配置事务管理器-->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
      <property name="dataSource" ref="dataSource"/> 
  </bean>
  
  <!-- 开启事务注解，当配置的事务管理器的id为transactionManager时，transaction-manager属性可以省略，如果不是，则必须显示赋值-->
  <tx:annotation-driven transaction-manager="txManager"/>
  ```

  

  * xml配置

  ```xml
  <!-- 配置数据源-->
  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close"> 
      <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
      <property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
      <property name="username" value="scott"/> 
      <property name="password" value="tiger"/> 
  </bean> 
  
  <!-- 配置事务管理器-->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
      <property name="dataSource" ref="dataSource"/> 
  </bean>
  
  <!-- 配置普通的bean-->
  <bean id="fooService" class="x.y.service.DefaultFooService"/> 
  
  <!-- 配置事务管理与AOP关联-->
  <tx:advice id="txAdvice" transaction-manager="txManager"> 
      <tx:attributes> 
          <tx:method name="get*" read-only="true"/> 
          <tx:method name="*"/> 
      </tx:attributes> 
  </tx:advice> 
  
  <!-- 配置AOP-->
  <aop:config> 
      <aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
      <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
  </aop:config>
  ```

  

**例子源码地址：
https://github.com/LDknife/spring-example/tree/master/spring-transaction**



### 5、Spring 事务管理的属性配置

* 值(value)

  标识属于哪个事务管理器。如果只有一个事务管理器，默认可以不用设置。

* 传播行为(propagation)

  当事务方法被另一个事务方法调用时，必须指定事务是如何传播的。例如：方法可能继续使用现有的事务，也可能开启一个新的事务。

  Spring定义了7种传播行为：

| 属性值        | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| REQUIRED      | （默认值）如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行 |
| REQUIRED_NEW  | 当前的方法必须启动新事务，并在它自己的事务内运行。如果有事务正在运行，应该将它挂起。 |
| SUPPORTS      | 如果有事务在运行，当前的方法就在这个事务内运行。否则它可以不运行在事务中。 |
| NOT_SUPPORTED | 当前的方法不应该运行在事务中，如果有运行的事务，将他挂起。   |
| MANDATORY     | 当前的方法必须运行在事务内部，如果没有正在运行的事务，就抛出异常。 |
| NEVER         | 当前的方法不应该运行在事务中，如果有运行的事务，就抛出异常。 |
| NESTED        | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行。否则，就启动一个新的事务，并在它自己的事务内运行。 |

* 隔离级别(isolation)

  当多个事务并发执行时，会引发一些问题：

  * 脏读：
    当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
  * 不可重复读：
    指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
  * 幻读：
    幻读与不可重复读类似。它发生在一个事务读取了几行数据，接着另一个并发事务插入了一些数据时。在随后的查询中，第一个事务就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

  事务的隔离级别：为解决上述问题，引入了事务隔离的概念。隔离级别越高，数据一致性就越好，但并发性越弱。
  SQL标准定义了四种隔离级别：

  * **READ_UNCOMMITTED（读未提交）：**
    值为1，最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、不可重复读或者幻读。**
  * **READ_COMMITTED（读已提交）：**
    值为2，允许读取并发事务已经提交的数据，**可以阻止脏读，但是不可重复读或者幻读仍有可能发生。**

  * **REPEATABLE_READ（可重复读）：**
    值为4，对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但是幻读仍有可能发生。** mysql默认的隔离级别。

  * **SERIALIZABLE（串行化）：**
    值为8，最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，**该级别可以防止脏读、不可重复读和幻读。**

* 触发事务回滚的异常

  默认情况下捕获到RuntimeException或Error时回滚。
  也可以设置属性：

  * rollbackFor：指定必须进行回滚的异常类型，可以为多个。
  * rollbackForClassName：指定必须进行回滚的异常类全类名，可以为多个。
  * noRollbackFor：指定不进行回滚的异常类型，可以为多个。
  * noRollbackForClassName：指定不进行回滚的异常类全类名，可以为多个。

* 超时(timeout)

  事务在强制回滚之前可以保持多久。这样可以防止长期运行的事务占用资源。默认为 -1。

* 只读(readOnly)

  标识这个事务只是读取数据，不更新数据。默认为false。