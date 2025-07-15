# 面试题汇总
> 无法跳转到题目请在右侧目录索引中输入对应题目的序号点击即可


## 1. Java常用集合   HashMap和ConcurrentMap区别

HashMap 和 ConcurrentMap（通常是 ConcurrentHashMap）的区别主要有三点：

线程安全：HashMap 是非线程安全的，不能在多线程下直接用；ConcurrentMap 是为并发设计的，支持多线程安全访问。

实现机制：ConcurrentMap 内部用的是CAS + 局部锁（JDK8 起），效率比 Hashtable 更高，避免了全表加锁。

null 支持：HashMap 允许一个 null 键、多个 null 值；ConcurrentMap 不允许 null 键也不允许 null 值。

总结一句话：单线程用 HashMap，多线程用 ConcurrentHashMap。


## 2. JVM内存模型   方法区存什么

方法区是 JVM 内存的一部分，主要用来存类的信息和静态内容，包括：

- 类的结构信息：类名、父类名、接口、字段、方法等；

- 常量池：字符串常量、类/方法符号引用等；

- 静态变量：static 修饰的字段；

- JIT 编译后的代码（部分 JVM 实现）；

- 运行时常量池。

补充一句（视时间而定）：
在 JDK8 之后，方法区由永久代（PermGen）改成了元空间（Metaspace），存储在本地内存，不再使用堆内存。

一句话总结：
方法区存类相关的元数据、常量池和静态变量，不是线程私有的内存。

## 3. Spring和SpringBoot的区别

**Spring 和 Spring Boot 的区别：**

1. **Spring 是框架，Spring Boot 是工具。**

   * **Spring** 是一个重量级的 Java 开发框架，提供了 IOC、AOP、事务等核心功能，但**需要手动配置**，比如 XML、注解、web.xml。
   * **Spring Boot** 是基于 Spring 的快速开发工具，**简化配置、开箱即用**，内置了 Tomcat、自动装配等功能。

2. **配置方式不同：**

   * Spring 配置繁琐，依赖较多。
   * Spring Boot 几乎“零配置”，用 `application.yml/properties` 统一管理。

3. **启动方式不同：**

   * Spring 项目需要部署到外部 Tomcat。
   * Spring Boot 可直接运行，内置服务器，像运行一个普通 Java 程序一样启动。

一句话总结：

**Spring 是基础框架，Spring Boot 是快速开发的增强工具，帮我们更快更简单地用好 Spring。**

## 4. IOC是什么，Bean是怎么初始化的

IOC 是什么？

\*\*IOC（控制反转）\*\*是 Spring 的核心思想，意思是：

> **对象的创建和依赖管理不再由我们手动 new，而是由 Spring 容器来负责。**

也叫 **依赖注入（DI）**，常见方式有：

* 构造方法注入
* set 方法注入
* 注解注入（如 `@Autowired`）

那 Bean 是怎么初始化的？

Spring 容器创建 Bean 大致经历这几个步骤：

1. **扫描配置**：根据配置或注解（如 `@ComponentScan`）找到需要托管的类；
2. **实例化**：通过反射创建对象（默认是单例）；
3. **依赖注入**：给 Bean 的字段/构造器注入依赖；
4. **初始化回调**（可选）：执行 `@PostConstruct`、`InitializingBean` 接口等；
5. **放入容器**：容器管理这个 Bean，随时可以通过 `@Autowired` 或 `ApplicationContext.getBean()` 获取。

一句话总结：

**IOC 就是把对象创建和依赖交给 Spring 管理，Bean 的初始化过程包括扫描、实例化、注入、初始化和管理五步。**

## 4. Mysql的隔离级别  幻读有没有在mysql被解决

**MySQL 有哪些隔离级别？**

MySQL（InnoDB 引擎）支持四种事务隔离级别，从低到高是：

1. **Read Uncommitted（读未提交）**：会出现脏读、不可重复读、幻读；
2. **Read Committed（读已提交）**：解决脏读，但**仍有不可重复读、幻读**；
3. **Repeatable Read（可重复读）**：解决脏读、不可重复读，**默认隔离级别**；
4. **Serializable（可串行化）**：最强，完全解决幻读，但性能差。

**幻读在 MySQL 中被解决了吗？**

**答案是：在 InnoDB 的 Repeatable Read 隔离级别下，MySQL 解决了幻读。**

* MySQL 通过 **间隙锁（Gap Lock）+ 临键锁（Next-Key Lock）** 机制，防止在范围查询中插入新记录，从而避免幻读。
* 所以虽然标准 SQL 定义幻读只能在 Serializable 被解决，但 **MySQL 的 Repeatable Read 实际就能避免幻读**。

一句话总结：

**MySQL 默认的 Repeatable Read 隔离级别已经解决了幻读，是通过“间隙锁”实现的。**




