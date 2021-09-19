---
title: Spring配置Bean
date: 2020-01-05 15:09:27
categories:
	- Spring
tags: 
	- Spring
toc: true
typora-root-url: Spring配置Bean
---



### 1、Spring IoC 容器简介
控制反转（IoC） 也称为依赖注入（DI）。其思想是反转资源的获取方向，spring容器在创建bean时通过构造方法，工厂方法定义其属性和其依赖项。



### 2、ApplicationContext

org.springframework.context.ApplicationContext接口代表Spring IoC容器，并负责实例化，配置和组装Bean。BeanFactory为容器的基本实现，而ApplicationContext是BeanFactory的子接口，增加了更多针对企业的高级功能。

主要实现类和接口：
* **ClassPathXmlApplicationContext**：从类路径下加载配置文件。

* **FileSystemXmlApplicationContext**：从文件系统中加载配置文件。

* **ConfigurableApplicationContext**：扩展ApplicationContext，新增 **refresh()** 和 **close()** 两个主要方法，让ApplicationContext具有启动，刷新和关闭上下文的能力。

* **WebApplicationContext**：用于web项目。



### 3、配置Bean

Spring 配置Bean的三种方式：
* 基于xml文件配置

* 基于注解配置

* 基于java+注解配置



### 4、依赖注入方式（基于xml文件配置）

Spring 支持三种依赖注入方式：
* 属性注入（常用）

* 构造器注入（常用）
* 工厂方法注入
* 实现FactoryBean<T>接口

#### 4.1 属性注入
通过setter方法注入Bean的属性值或依赖对象。
例子：

```xml
<bean id="helloWorld" class="com.ld.spring.HelloWorld">
       <property name="name" value="ld"/>
       <property name="age" value="18"/>
       <property name="sex" value="0"/>
       <property name="address">
            <value>中国</value>
       </property>
</bean>
```
在xml配置文件中：

* <bean>标签表示一个Bean

  * id：在IoC容器中必须唯一，若id没有指定，Spring自动将类名作为Bean的id。

  * class：全类名，通过映射的方式在IoC容器中创建Bean,所以要求Bean中**必须有无参的构造器**。



* <property>标签表示一个Bean的属性

  * name：表示属性名称

  * value或<value>子节点：表示属性值



#### 4.2 构造器注入

通过构造器注入Bean的属性值或依赖对象。
例子：

```xml
<bean id="helloWorld" class="com.ld.spring.HelloWorld">
       <constructor-arg value="ld" index="0"/>
       <constructor-arg value="18" name="age"/>
       <constructor-arg value="0" type="int"/>
       <constructor-arg index="3">
            <value>中国</value>
       </constructor-arg>
</bean>
```
在xml配置文件中：

<constructor-arg>标签表示构造器中的一个参数。
在构造器的参数中不存在歧义时，通过解析参数的类型进行匹配注入。
在构造器的参数中存在歧义时，通过解析显示指定的**参数类型，参数索引，参数名称**进行匹配注入。（注意：参数索引从0开始）



#### 4.3 工厂方法注入

1. 静态工厂方法

   使用静态方法的方式创建对象。
   例子：

```java
public class StaticFactory {    

    private static StaticFactory staticFactory = new StaticFactory();    
    
    private StaticFactory(){}    
    
    public static StaticFactory createInstance(){       
        return staticFactory;    
    }    
    
    public void test(){        
        System.out.println("我获取到实例对象了");    
    }
}
```
```xml
<bean id="staticFactory" class="com.ld.spring.bean.StaticFactory" factory-method="createInstance"/>
```
在xml配置文件中：

* factory-method：指向静态工厂的静态方法。



2. 实例工厂方法

使用实例工厂的实例对象，调用对象中的非静态方法来获取或创建其他类的实例对象。
例子：

```java
public class ClientService{
    public ClientService(){}
}
```
```java
public class DefaultServiceLocator { 

    private static ClientService clientService = new ClientServiceImpl(); 
    
    public ClientService createClientServiceInstance() { 
        return clientService; 
    } 
}
```
```xml
<!-- 实例工厂bean-->
 <bean id="serviceLocator" class="examples.DefaultServiceLocator"/> 
 <!-- 调用工厂方法获取其他bean-->
 <bean id="clientService" factory-bean="serviceLocator" factory-method="createClientServiceInstance"/>
```
在xml配置文件中：

* factory-bean：引入实例工厂的bean。

* factory-method：指向实例工厂的非静态方法。

  **注意：一个实例工厂里可以有一个或多个工厂方法。**



#### 4.4 实现FactoryBean<T> 接口

Spring中有两种类型的bean，一种是普通bean，另一种是工厂bean，即FactoryBean。
工厂bean跟普通的bean不同，其返回的对象不是指定类的一个实例，其返回的是该工厂bean的getObject方法所返回的对象。

实现FactoryBean接口，配置xml，注入bean到IoC容器。

1. Spring提供的FactoryBean接口：
```java
package org.springframework.beans.factory;

import org.springframework.lang.Nullable;

public interface FactoryBean<T> {    
    @Nullable    
    T getObject() throws Exception;   
    
    @Nullable    
    Class<?> getObjectType();    
    
    default boolean isSingleton() {        
        return true;    
    }
}
```
2. 实现FactoryBean接口：

自定义Car类

```java
public class Car{
    private String name;
    private double price;
    
    public Car(String name,double price){    
        this.name = name;    
        this.price = price;
    }
}
```
自定义CarFactoryBean类，实现FactoryBean接口

```java
public class CarFactoryBean implements FactoryBean<Car>{
    private String name;
    private double price;
    
    public CarFactoryBean(String name,double price){    
        this.name = name;    
        this.price = price;
    }
    
    public CarFactoryBean() {}
    
    @Override
    public Car getObject() throws Exception {    
        return new Car(name,price);
    }
    
    @Override
    public Class<?> getObjectType() {    
        return Car.class;
    }
}
```
3. 配置xml：
```xml
<bean id="car" class="com.ld.spring.bean.CarFactoryBean">    
    <constructor-arg value="audi"/>    
    <constructor-arg value="300000"/>
</bean>
```



### 5、依赖注入细节

* 字面量

支持字面量值的注入方式，若字面量值中包含特殊字符，可以使用 **<![CDATA[字面量值]]>** 把字面量值包裹起来。

```xml
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="<![CDATA[<ld>]]>"/>
</bean>
```
* null

可以使用专用的 **<null/>** 标签为Bean的字符串或其他对象类型的属性注入null值。

```xml
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld"/>
    <property name="car"><null/></property> 
</bean>
```
* 给bean的级联属性赋值。

* 使用**ref**属性注入外部已声明的bean。
```xml
<bean id="car" class="com.ld.spring.bean.Car">
    <property name="name" value="audi"/>
    <property name="price" value="300000"/>
</bean>

<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld"/>
    <property name="car" ref="car"/>
</bean>
```
* 内部bean。
```xml
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld"/>
    <property name="car">
        <bean class="com.ld.spring.bean.Car">
            <property name="name" value="bwm"/>
            <property name="price" value="400000"/>
        </bean>
    </property>
</bean>
```
* 集合属性
```xml
<!-- <list> 标签-->
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld">
    <property name="addressList">
        <list>
            <value>北京</value>
            <value>上海</value>
            <value>广州</value>
        </list>
    </property>
</bean>

<!-- <set>标签-->
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld">
    <property name="cars">
        <set>
            <ref bean="car1"/>
            <ref bean="car2"/>
        </set>
    </property>
</bean>

<!-- <map>标签-->
<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld">
    <property name="carMaps">
        <map>
            <entry key="AA" vlaue-ref="car1"/>
            <entry key="BB" vlaue-ref="car2"/>
        </map>
    </property>
</bean>

<!-- <props>标签-->
<bean id="dataSourc" class="com.ld.spring.bean.DataSource">
    <property name="properties">
        <props>
            <prop key="user">root</prop>
            <prop key="password">123456</prop>
            <prop key="url">jdbc:mysql://127.0.0.1:3306/test</prop>
            <prop key="driver">com.mysql.jdbc.Driver</prop>
        </props>
    </property>
</bean>
```
* 使用 utility scheme 定义集合
```xml
<util:list id="cars">
    <ref bean="car"/>
    <ref bean="car1"/>
</util:list>

<bean id="person" class="com.ld.spring.bean.Person">
    <property name="name" value="ld"/>
    <property name="cars" ref="cars"/>
</bean>
```
* 使用p命名空间

Spring从2.5版本引入新的p命名空间，用于简化xml配置。

```xml
<bean id="car" class="com.ld.spring.bean.Car" p:name="audi" p:price="300000"/>
```



### 6、Bean的高级配置

* Bean的继承

Spring允许继承bean的配置，被继承的bean称为父bean，继承父bean的bean为子bean。
* 子bean可以继承父bean的配置，包括属性配置。

* 子bean也可以覆盖从父bean继承过来的配置。

* 可以设置父bean为模板，即设置<bean>的abstract属性为true。Spring不会实例化这个bean。

* 可以不为父bean设置class属性，则这个bean必定为抽象bean,即abstract属性为true。

* 并不是所有的属性都可以被继承，比如：autowire,abstract 等。

```xml
<bean id="dept" class="com.ld.spring.bean.Dept">
    <property name="deptId" value="1001"/>
    <property name="deptName" value="PM"/>
</bean>

<bean id="emp01" class="com.ld.spring.bean.Emp">
    <property name="id" value="01"/>
    <property name="name" value="ld"/>
    <property name="dept" ref="dept">
</bean>

<bean id="emp02" parent="emp01">
    <property name="id" value="02"/>
    <property name="name" value="monkey"/>
</bean>
```

* Bean之间的依赖

有时创建一个bean需要保证另一个bean也被创建，这时我们称前面的bean对后面的bean有依赖。**依赖也可以不引用。**

```xml
<bean id="dept" class="com.ld.spring.bean.Dept">
    <property name="deptId" value="1001"/>
    <property name="deptName" value="PM"/>
</bean>
<!-- emp 依赖 dept-->
<bean id="emp" class="com.ld.spring.bean.Emp" depends-on="dept">
    <property name="id" value="01"/>
    <property name="name" value="ld"/>
</bean>
```

* Bean的懒加载

SpringIoC容器中默认都为单例Bean,即在创建IoC容器时，创建所有的Bean。有时我们为了性能不需要预先实例化bean，所以Spring提供延迟初始化Bean的方式，即首次请求时创建。**设置lazy-init为true即可。**

```xml
<bean id="person" class="com.ld.spring.bean.Person" lazy-init="true"/>
```
我们也可以在容器级别控制延迟初始化。

```xml
<beans default-lazy-init="true">
    ...
</beans>
```



### 7、Bean的作用域

在Spring中，可以在 **<Bean>标签的scope属性里设置bean的作用域**。
Spring支持的作用域：

* singleton：(默认值) 在Spring IoC容器中所有bean仅存在单个对象实例。**在创建IoC容器时，创建定义的bean对象**。

* prototype：可以将bean创建任意数量的对象实例。**在调用bean时，才创建bean对象**。

* request：在WEB容器中使用，将bean对象的作用域限定在单个HTTP请求的生命周期中。

* session：在WEB容器中使用，将bean对象的作用域限定在单个Session的生命周期中。

* application：在WEB容器中使用，将bean对象的作用域限定在ServletContext的生命周期中。

* websocket：在WEB容器中使用，将bean对象的作用域限定在websocket的生命周期中。



### 8、Bean的生命周期

Bean的生命周期流程：
* 通过构造器或工厂方法创建bean实例。

* 为bean的属性赋值和对其他bean的引用。

* 调用bean的初始化方法。

* 使用bean对象。

* 当容器关闭时，调用bean的销毁方法。

注意：在配置bean时，可以通过init-method和destory-method属性指定初始化和销毁方法。
**例子源码地址：
https://github.com/LDknife/spring-example/tree/master/spring-bean**

![](bean生命周期.PNG)



### 9、Bean的后置处理器

bean的后置处理器允许在调用bean的初始化方法的前后对bean进行额外处理。
bean的后置处理器是对IoC容器中所有的bean实例逐一处理，而非单一实例。其典型应用是：检查bean属性的正确性或根据特定的标准更改bean的属性。

Spring提供的BeanPostProcessor接口：

```java
package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;
import org.springframework.lang.Nullable;

public interface BeanPostProcessor {    
    @Nullable    
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws                                                                                  BeansException {        
        return bean;    
    }    
    @Nullable    
    default Object postProcessAfterInitialization(Object bean, String beanName) throws                                                                                      BeansException {       
        return bean;    
    }
}
```



### 10、引入外部配置文件

当xml配置文件中配置的bean比较多时，查找或者修改一些bean的配置会变得比较困难。因此Spring提供将bean的配置信息以properties文件存储，然后以引入外部配置文件的方式，使其关联起来。从而实现修改properties文件即可修改bean的配置信息。

**前提：xml配置文件中需要引入context命名空间。**

1. 创建并配置properties文件。
```properties
jdbc.username=root
jdbc.password=123456
jdbc.url=jdbc:mysql://127.0.0.1:3306/test
jdbc.driver=com.mysql.jdbc.Driver
```

2. 在xml配置文件中引入properties属性文件。
* 方式一（常用）：
```xml
<!-- classpath:xxx 表示属性文件位于类路径下-->
<!-- 如果有多个properties配置文件需要引入，那么一定要加上 ignore-unresolvable="true",多个配置文件以”,“分割-->
<context:property-placeholder location="classpath:jdbc.properties"/>
```
* 方式二：
```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:jdbc.properties" />
</bean>
```

3. 在xml配置文件中关联properties属性
```xml
<!-- ${} 用于关联properties属性-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="driverClass" value="${jdbc.driver}"/>
</bean>
```



### 11、自动装配

根据指定的装配规则，不需要明确指定，Spring自动将匹配的属性值注入bean中。
Spring提供的装配模式：

* byName：将目标bean的名称和属性名完全相同才可装配成功。

* byType：将类型匹配的目标bean,作为属性注入到另一个bean中。如果目标bean存在多个，将无法判定使用哪一个bean。无法装配成功。

* constructor：当bean中存在多个构造器时，此种自动装配方式将会很复杂。不推荐使用。

**注意：在xml中自动装配比较笨拙，开发时多用于注解的方式。**



### 12、基于注解配置Bean

相对于xml方式而言，通过注解的方式配置bean更加简洁和优雅，是开发中常用的使用方式。

#### 12.1、常用组件注解
* @Component：标识一个受SpringIoC容器管理的组件。
* @Controller：标识一个受SpringIoC容器管理的表述层控制器组件。
* @Service：标识一个受SpringIoC容器管理的业务逻辑层组件。
* @Repository：标识一个受SpringIoC容器管理的持久化层组件。



#### 12.2、组件命名规则

* 默认情况：使用组件的简单类名首字母小写后得到的字符串作为bean的id。
* 自定义：可以自定义bean的id。
注意：使用组件注解的value属性指定bean的id。



#### 12.3、扫描组件

组件被注解标识后，需要通过Spring进行扫描才能够检测到。
指定被扫描的package（xml文件需要引入context命名空间）：

* base-package 属性可以指定一个需要扫描的基类包，Spring容器将会扫描这个基类包及其子包中的所有类。

* 当需要扫描多个包时可以使用逗号分隔。

* 如果仅希望扫描特定的类，而非基包下的所有类，可以使用 resource-pattern 属性过滤特定的类。

* <context:include-filter />子节点表示要包含的目标类。注意：通常需要与 use-default-filters 属性配合使用才能达到效果。即：通过将 use-default-filters  属性设置为false，禁用默认过滤器，然后扫描的就只是include-filter中的规则指定的组件了。

* <context:exclude-filter />子节点表示要排除的目标类。

* component-scan 下可以拥有若干个 include-filter 和 exclude-filter 子节点。

* 过滤表达式：
  1. annotation：过滤所有标注此类注解的类。
  2. assignable：过滤所有同父类的子类。

```xml
<context:component-scan base-package="com.ld.spring"/>
```



#### 12.4、组件装配

在指定要扫描的包时，<context:component-scan >标签会自动注册一个bean的后置处理器：AutowiredAnnotationBeanPostProcessor 的实例。该后置处理器可以自动装配标记了@Autowired、@Resource 或 @Inject 注解的属性。

* @Autowired：
  1. 根据类型实现自动装配。
  2. 构造器、普通字段（即使是非public）、一切具有参数的方法都可以应用@Autowired注解。
  3. 默认情况下，所有使用@Autowired 注解的属性都需要被设置。当Spring找不到匹配的bean装配属性时，会抛出异常。
  4. 若某一属性允许不被设置，可以设置@Autowired注解的 **required 属性为false。**
  5. 默认情况下，当IoC容器里存在多个类型兼容的bean时，Spring会尝试匹配bean的id值是否与变量名相同，如果相同则进行装配。如果bean的id值不相同，通过类型的自动装配将无法工作。此时可以在 **@Qualifier 注解里提供bean的名称**。Spring 甚至允许在方法的形参上标注@Qualifier注解以指定注入bean的名称。
  6. @Autowired 注解可以应用在数组类型的属性上，此时Spring 将会把所有的匹配的bean进行自动装配。
  7. @Autowired 注解可以应用在集合属性上，此时Spring 读取该集合的类型信息，然后自动装配所有与之兼容的bean。
  8. @Autowired 注解可以应用在java.util.Map上，若该map的键值为String，那么Spring 将自动装配与值类型兼容的bean作为值，并以bean的id值作为键。

* @Resource

  @Resource 注解要求提供一个bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为bean的名称。

* @Inject

  @Inject 和 @Autowired 注解一样也是按类型装配的，但没有required 属性。