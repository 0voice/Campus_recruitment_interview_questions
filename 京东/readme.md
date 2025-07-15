# 面试题
> 无法跳转到题目请在右侧目录索引中输入对应题目的序号点击即可

## 1. 分布式锁、Redisson 的实现

**分布式锁实现方式：**

* **基于 Redis：**

  * `SET key value NX PX 30000`（原子加锁 + 过期时间）
  * 释放锁时验证 value，确保是自己的锁

* **基于 Zookeeper：**

  * 创建临时顺序节点 + 监听前一个节点，公平性好，适用于强一致场景

**Redisson 实现原理：**

* 使用 Redis + Lua 脚本 + watchdog 自动续期
* 支持可重入、公平锁、读写锁、联锁等功能
* 默认加锁时间 30s，可通过 watchdog 自动续约，防止死锁


## 2. Redis 的数据结构有哪些？底层是什么？

| Redis 类型    | 底层结构                | 应用场景       |
| ----------- | ------------------- | ---------- |
| String      | 简单动态字符串 SDS         | 计数、缓存、分布式锁 |
| List        | 双向链表 / QuickList    | 消息队列、任务列表  |
| Set         | Hash 表 / IntSet     | 去重集合、标签    |
| Hash        | ZipList / HashTable | 用户信息、对象存储  |
| ZSet        | SkipList + HashMap  | 排行榜、分数排序   |
| Bitmap      | 位图                  | 活跃统计、布隆过滤器 |
| HyperLogLog | 基数统计                | UV 统计      |


## 3. JUC 中的 `synchronized` 与可重入锁（`ReentrantLock`）的区别？

| 对比项  | `synchronized` | `ReentrantLock`      |
| ---- | -------------- | -------------------- |
| 锁类型  | 内置锁，隐式获取释放     | 显式锁，需手动 unlock       |
| 可中断  | 否              | 是                    |
| 是否公平 | 否（非公平）         | 可选公平锁（性能略差）          |
| 可重入  | 是              | 是                    |
| 条件变量 | 不支持            | 支持 `Condition` 多条件等待 |

> **建议：** 简单用法用 `synchronized`，复杂控制用 `ReentrantLock`

## 4. JVM 内存结构与 GC 机制？

**JVM 运行时内存结构：**

* 堆（Heap）：对象实例，GC管理的重点
* 方法区（元空间）：类的结构信息
* 虚拟机栈：局部变量表、操作栈
* 本地方法栈：调用 native 方法
* 程序计数器：线程私有，当前执行字节码位置

**GC机制：**

* 判断对象是否可达（可达性分析）
* 回收算法：标记清除、复制算法、分代收集
* 常见 GC 收集器：Serial、CMS、G1、ZGC、Shenandoah
* **GC 触发场景：** 内存不足 / System.gc() / Full GC


## 5. MySQL 的使用场景？说说你的使用经验？

**使用场景：**

* OLTP 场景（高频读写业务数据）
* 互联网中小型业务系统，如电商、CMS、订单系统

**经验点：**

* 使用 InnoDB 引擎，支持事务、行级锁
* 创建合适索引、避免索引失效
* 慢 SQL 分析（EXPLAIN + 慢日志）
* 主从复制、读写分离、高可用部署（MGR、Keepalived）


## 6. 什么是聚集索引和非聚集索引？

| 类型    | 描述                     |
| ----- | ---------------------- |
| 聚集索引  | 数据行按索引顺序存储，一个表只有一个（主键） |
| 非聚集索引 | 索引存主键指针，数据单独存储，需回表查询   |

> **回表：** 非聚集索引查找时，只能找到主键值，需再回主键索引查数据


## 7. B+ 树是如何工作的？

* 多路平衡树（如 16叉、32叉）
* 所有数据只存储在 **叶子节点**，非叶子节点只做索引
* 叶子节点通过**链表相连**，支持**范围查询**

**优点：**

* 层数少，减少磁盘 IO
* 顺序存储叶子节点 → 范围查询快
* 每次查找从根节点开始，二分查找向下走


## 8. 事务的 ACID 特性和隔离级别有哪些？

**ACID 四特性：**

1. **原子性（A）**：要么全部成功，要么全部失败
2. **一致性（C）**：前后一致，如库存+订单同时变化
3. **隔离性（I）**：事务互不干扰
4. **持久性（D）**：提交后数据永久保存

**MySQL 四种隔离级别（InnoDB 支持）：**

* Read Uncommitted（脏读）
* Read Committed（不可重复读）
* Repeatable Read（防幻读，MySQL 默认）
* Serializable（最安全最慢）


## 9. 乐观锁和悲观锁的原理与适用场景？

**乐观锁：**

* 假设不会冲突，提交时检查（如版本号 CAS）
* **适合读多写少场景**

**悲观锁：**

* 假设会冲突，操作前先加锁（如 `select for update`）
* **适合写多读少场景**


## 10. 为什么使用 Spring？Spring Boot 有哪些优势？你用过 Spring 全家桶的哪些组件？

**为什么用 Spring：**

* 降低耦合、方便管理 Bean（IOC + AOP）
* 提供事务、缓存、MVC、消息队列、数据库等集成支持

**Spring Boot 优势：**

* 自动配置、内嵌 Tomcat、快速开发
* 减少配置（约定优于配置）
* 更好支持微服务（Spring Cloud）

**常用全家桶组件：**

* Spring MVC、Spring Data JPA、Spring Security
* Spring AOP、Spring Cache、Spring Test


## 11. Spring 的 AOP 是什么？

**AOP（面向切面编程）：**

* 在不修改源代码情况下给程序添加通用逻辑，如日志、权限、事务


## 12. 介绍一下 AOP

\*\*AOP（面向切面编程）\*\*是 Spring 提供的一种功能增强方式，用于在不改业务逻辑代码的情况下，**在方法前后插入通用逻辑**（如日志、权限、事务等）。
Spring AOP 基于 **代理机制**（JDK 动态代理 或 CGLIB）实现。


## 13. Spring 底层实现机制了解吗？

Spring 核心原理：

* **IOC 容器**：通过 BeanFactory/ApplicationContext 加载配置，管理对象生命周期
* **AOP**：基于代理实现方法增强（使用 JDK 动态代理或 CGLIB）
* **事件机制**：基于发布-订阅模式
* **自动装配**：通过反射、注解实现依赖注入


## 14. 什么是动态代理？

动态代理是运行时生成的代理类，可以在不修改目标类的情况下增强功能。

* **JDK 动态代理**：要求接口，基于 `InvocationHandler`
* **CGLIB 代理**：基于继承和字节码增强，不需要接口（生成子类）

Spring AOP 会根据是否有接口自动选择 JDK 或 CGLIB。


## 15. Spring 框架中用到了哪些设计模式？

| 模式名称  | 应用场景说明                            |
| ----- | --------------------------------- |
| 单例模式  | 默认 Bean 是单例                       |
| 工厂模式  | BeanFactory、FactoryBean           |
| 策略模式  | `@Qualifier` 指定不同实现               |
| 模板方法  | JdbcTemplate、RestTemplate 等       |
| 代理模式  | AOP 动态代理实现                        |
| 观察者模式 | 事件发布机制（ApplicationEventPublisher） |

## 16. Spring 是怎么实现事务控制的？

* 使用 `@Transactional` 注解 + AOP 切面
* 本质上是 AOP 拦截方法 → 开启事务 → 执行方法 → 提交/回滚事务
* 底层使用 `TransactionManager` 管理事务，常见的如 `DataSourceTransactionManager`



## 17. Redis 用途有哪些？

* 缓存（热点数据、session 缓存）
* 分布式锁（setnx + lua）
* 排行榜（zset）
* 限流（滑动窗口、令牌桶）
* 消息队列（list/stream）
* 计数器、签到等（bitmap/hyperloglog）


## 18. 方法重载和覆写的区别？

* **重载（Overload）**：同类中方法名相同，参数不同（编译时多态）
* **覆写（Override）**：子类重写父类方法，必须方法名/参数一致（运行时多态）


## 19. 堆和栈的区别？

| 对比项  | 栈                | 堆         |
| ---- | ---------------- | --------- |
| 存储内容 | 基本类型 + 引用 + 局部变量 | 对象实例      |
| 管理方式 | 由编译器自动管理         | JVM GC 管理 |
| 生命周期 | 随方法执行完就释放        | 对象可长时间存在  |
| 性能   | 更快               | 较慢（涉及 GC） |


## 20. List 有哪些实现类，分别有什么特点？

| 实现类                  | 特点                    |
| -------------------- | --------------------- |
| ArrayList            | 底层数组，查询快，增删慢（扩容耗时）    |
| LinkedList           | 底层双向链表，插入删除快，查询慢      |
| Vector               | 线程安全，但性能差             |
| Stack                | 继承 Vector，实现 LIFO 栈结构 |
| CopyOnWriteArrayList | 线程安全，读多写少场景           |


## 21. HashMap 扩容机制？

* 默认初始容量 16，负载因子 0.75
* 当 size > threshold 时，**扩容为原来的 2 倍**
* 扩容时会**重新哈希所有元素**
* Java 8 后，当链表长度 > 8 且数组长度 > 64，会转为红黑树结构

## 22. JVM 启动参数用过吗？

常用的 JVM 启动参数：

```bash
-Xmx512m     # 最大堆内存
-Xms512m     # 初始堆内存
-Xss256k     # 每个线程的栈大小
-XX:+PrintGCDetails   # 打印 GC 日志
-XX:+UseG1GC          # 使用 G1 收集器
```



## 23. 数据库索引怎么使用？

* 给**频繁查询、排序、join 的字段加索引**
* 使用联合索引时遵循**最左匹配原则**
* 不要给频繁更新的字段加索引（会增加写开销）
* 用 `EXPLAIN` 分析 SQL 是否用到了索引

## 24. 数据库表怎么设计的？

设计数据库表的建议：

* 符合**三范式**（减少冗余）
* 主键使用自增 ID 或雪花 ID
* 字段类型合适（如金额用 decimal，时间用 datetime）
* 加入必要的索引
* 保留扩展字段（如 ext\_json、status 枚举）

## 25. 为什么有了基本类型还需要封装类？

Java 是面向对象语言，封装类（如 `Integer`）的意义是：

* 提供更多方法（如 `Integer.parseInt()`、`compareTo()` 等）
* 可以存入集合类（如 `List<Integer>`，因为集合只存对象）
* 支持 `null` 值，适合数据库字段
* 用于泛型场景，基本类型不能作为泛型参数


## 26. Java 内部类和匿名内部类的作用？

**内部类（成员/静态/局部）：**

* **封装逻辑**，防止被外部访问
* 可以访问外部类的成员变量
* 用于类与类之间的**强依赖关系**

**匿名内部类：**

* 用于**一次性重写父类或接口的方法**（如线程、监听器）
* 简洁，通常用于回调逻辑或简化代码



## 27. Java 支持运算符重载吗？

**不支持**，Java 为了代码简洁和可读性，**禁止开发者自定义运算符重载**。
只有少数内建类型（如 `String` 的 `+` 拼接）在语法上做了重载支持。


## 28. 如果让你实现菱形继承，你怎么做？

Java **类不支持多继承**，为了避免菱形继承问题。

实现方式：**使用接口 +组合**，多个接口可以被一个类实现：

```java
interface A {}
interface B extends A {}
interface C extends A {}
class D implements B, C {}
```

若需要共享逻辑，可以用 **默认方法（default）+ 接口实现类组合注入**。


## 29. 常见的内存泄露场景？

* **静态变量持有对象引用**，导致对象不能被回收
* **集合类中引用未清理**（如 `Map` 未移除 key）
* **监听器/回调未注销**
* **ThreadLocal 未手动 remove()**
* **连接池/文件/Socket 等资源未关闭**
* **匿名内部类隐式引用外部类**


## 30. 怎么判断链表有无环，环的大小怎么确定？

**判断链表有环：**

使用**快慢指针**法：

* 慢指针每次走一步，快指针每次走两步
* 如果快慢指针相遇，说明有环；否则到 `null` 无环

**环的大小：**

* 相遇后，从相遇点开始继续走，直到再回到相遇点，走过的步数就是环的长度

**环的起点：**

* 再设置一个指针从头开始，和慢指针同时走一步，相遇点就是环的入口

## 31. DPO 和 PPO 的原理与异同？

* **DPO（Deterministic Policy Optimization）**：通常指确定性策略优化，比如 DDPG。策略是确定性的，给定状态直接输出动作。适合连续动作空间。

* **PPO（Proximal Policy Optimization）**：是一种**近端策略优化**算法，属于策略梯度方法，采用**概率策略**（概率分布采样动作）。通过限制更新步长（剪切概率比率）避免策略大幅度更新，保证训练稳定。

**异同：**

| 点    | DPO       | PPO        |
| ---- | --------- | ---------- |
| 策略类型 | 确定性策略     | 概率性策略      |
| 稳定性  | 较低，可能过度更新 | 采用裁剪，更新稳定  |
| 采样方式 | 动作输出直接    | 动作从概率分布采样  |
| 应用   | 连续动作控制    | 连续和离散动作均适用 |


## 32. 手撕题：数组中每个长度为 k 的滑动窗口内的最大值，要求最短时间复杂度？

**问题描述**：给定长度为 n 的数组，求每个长度为 k 的窗口中的最大值。

**最优解法（时间复杂度 O(n)）**：

* 使用**双端队列（Deque）**，队列存索引，保持队头是当前窗口最大元素索引。
* 遍历数组时：

  * 移除队尾比当前元素小的索引（保持队列单调递减）
  * 插入当前元素索引
  * 移除队头不在当前窗口范围内的索引
  * 当遍历到第 k 个元素后，每步输出队头元素对应值作为窗口最大值

**示例代码（Java）**：

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    if(nums == null || k <= 0) return new int[0];
    int n = nums.length;
    int[] res = new int[n - k + 1];
    Deque<Integer> deque = new LinkedList<>();
    for(int i = 0; i < n; i++) {
        while(!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }
        deque.offerLast(i);
        if(deque.peekFirst() == i - k) {
            deque.pollFirst();
        }
        if(i >= k - 1) {
            res[i - k + 1] = nums[deque.peekFirst()];
        }
    }
    return res;
}
```

## 33. 口述环形链表中环的长度

* 使用快慢指针法，快指针一次走两步，慢指针一次走一步。
* 如果快慢指针相遇，说明有环。
* 从相遇点开始，慢指针继续走，直到再次回到相遇点，走过的步数就是环的长度。

## 34. 口述快速排序（快排），时间复杂度

* 快速排序通过选取一个“基准”元素，把数组分成左右两部分，左边比基准小，右边比基准大，然后递归排序左右两部分。
* **平均时间复杂度**是 O(n log n)。
* 最坏情况下（每次选的基准都是最大或最小）时间复杂度退化为 O(n²)。


## 35. 隐马尔可夫模型（HMM）

* 是一种统计模型，描述一个隐藏的马尔可夫链通过观测变量产生观测序列的过程。
* 模型包含：状态转移概率、观测概率和初始状态概率。
* 常用来解决序列标注、语音识别等问题。



## 36. 知不知道 BM25 算法？

* BM25 是一种基于词频的文本相关度排序算法，用于信息检索。
* 它是 TF-IDF 的改进，考虑了词频饱和和文档长度的影响。
* 常用于搜索引擎对文档的相关性打分。

## 37. 知不知道 Attention 机制？

* Attention 是神经网络中的一种机制，允许模型在处理输入时“关注”不同部分，动态调整权重。
* 主要用于机器翻译、文本理解、图像识别等领域。
* Transformer 模型的核心就是多头自注意力机制（Multi-head Self-Attention）。

## 38. 手撕：反转链表

**思路**

* 使用三个指针：`prev`，`curr`，`next`。
* 遍历链表，每次将 `curr.next` 指向 `prev`，然后向后移动指针。
* 最后返回新的头节点 `prev`。

**代码示例（Java）**

```java
public ListNode reverseList(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```


## 39. 手撕：得到两个线程的计算成果

**思路**

* 创建两个线程分别计算各自任务。
* 使用 `FutureTask` 或 `Callable` 获取计算结果。
* 主线程调用 `future.get()` 等待两个线程结束并获取结果。

**代码示例（Java）**
```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Callable<Integer> task1 = () -> {
    // 计算任务1
    return 10;
};
Callable<Integer> task2 = () -> {
    // 计算任务2
    return 20;
};

Future<Integer> future1 = executor.submit(task1);
Future<Integer> future2 = executor.submit(task2);

int result1 = future1.get();
int result2 = future2.get();

int total = result1 + result2;
executor.shutdown();
```

## 40. 手撕：找到数组中重复的数字和出现次数

**思路**

* 使用 HashMap 统计每个数字出现次数。
* 遍历数组，更新计数。
* 最后遍历 HashMap 输出重复数字及次数。

**代码示例（Java）**

```java
public Map<Integer, Integer> findDuplicates(int[] nums) {
    Map<Integer, Integer> countMap = new HashMap<>();
    for (int num : nums) {
        countMap.put(num, countMap.getOrDefault(num, 0) + 1);
    }
    Map<Integer, Integer> duplicates = new HashMap<>();
    for (Map.Entry<Integer, Integer> entry : countMap.entrySet()) {
        if (entry.getValue() > 1) {
            duplicates.put(entry.getKey(), entry.getValue());
        }
    }
    return duplicates;
}
```






