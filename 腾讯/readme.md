# 面试题
> 无法跳转到题目请在右侧目录索引中输入对应题目的序号点击即可

## 1. Redis 和 MySQL 如何保证一致性？

**答法思路：最终一致性 + 双写问题控制：**

* 可使用**延迟双删**策略（写 MySQL 后删缓存，再延迟一次删）
* 或使用**Binlog + 消费队列异步更新缓存**（如 Canal）
* 更高级可用 **缓存更新消息队列 + 异步重建缓存**


## 2. DDD 是什么？聚合根、实体解释

**领域驱动设计（DDD）**：将系统按业务模型组织，提升复杂系统可维护性。

* **实体（Entity）**：有唯一标识（ID），生命周期长，如用户、订单
* **值对象（Value Object）**：无 ID、只关心值，如地址、金额
* **聚合（Aggregate）**：一组相关对象组成的业务一致性边界
* **聚合根（Aggregate Root）**：代表聚合的唯一入口，负责聚合内部规则和约束


## 3. 慢查询优化怎么做？

* 添加**合适索引**（走索引避免全表扫描）
* 避免 `SELECT *`，限制返回字段
* 拆分页/limit深分页，用索引覆盖
* 使用 `EXPLAIN` 分析执行计划
* 表结构优化、分库分表、预加载热数据


## 4. 什么是聚簇索引？

* 数据**物理顺序 = 索引顺序**，存储在一棵B+树中
* 每张表只能有一个聚簇索引（InnoDB 主键默认是）
* 非聚簇索引会存储主键作为“指针”



## 5. 联合索引在什么情况下会失效？

* **没有遵循最左前缀原则**
* 中间字段跳过查询
* 使用了函数/隐式类型转换
* 出现范围查询（如 `>`）后，右边字段索引失效


## 6. Redis 持久化方式？

* **RDB**：快照方式定期保存内存状态到磁盘
* **AOF**：将写操作日志持续追加到文件，重启时回放
* 可混合使用，兼顾恢复速度和数据完整性

## 7. 跳表是什么？

* 一种**有序链表+多级索引结构**，支持快速查找（O(logN))
* Redis SortedSet 内部结构就是跳表

## 8. 缓存雪崩 / 穿透 / 击穿 概念与解决方案

| 问题     | 描述                    | 解决方案                   |
| ------ | --------------------- | ---------------------- |
| **穿透** | 请求数据缓存和数据库都没有         | 缓存空值 / 布隆过滤器           |
| **击穿** | 热点 key 过期，瞬间大量请求打到数据库 | 分布式锁 / 提前刷新            |
| **雪崩** | 大量 key 同时过期           | 设置随机过期时间 / 多级缓存 / 限流降级 |


## 9. TCP 三次握手 + 四次挥手

* **三次握手：**

  1. 客户端：SYN
  2. 服务端：SYN-ACK
  3. 客户端：ACK
     → 确保双方都能发送和接收

* **四次挥手：**

  1. 客户端：FIN
  2. 服务端：ACK
  3. 服务端：FIN
  4. 客户端：ACK
     → 连接被双方优雅关闭


## 10. 拥塞窗口 vs 接收窗口

* **接收窗口（rwnd）**：接收方告诉发送方自己还能接收多少字节
* **拥塞窗口（cwnd）**：发送方根据网络拥塞状况控制的窗口，避免网络堵塞
* 真实发送窗口 = `min(rwnd, cwnd)`

## 11. 拥塞窗口何时变成一半？

* **发生丢包时**（超时或三次重复 ACK），TCP 进入 **快速重传/拥塞避免阶段**
* 拥塞窗口 `cwnd` 会减半，触发 **乘法减小算法（AIMD）**


## 12. 对 CDN 业务的了解

CDN（内容分发网络）核心是**就近加速**，通过把内容缓存到**各地边缘节点**，提升访问速度、减轻源站压力。

* 加速对象：静态资源（HTML/CSS/JS/图片/视频）为主
* 工作流程：DNS 解析调度 → 用户访问最近边缘节点 → 命中缓存或回源
* CDN 厂商：阿里云 CDN、腾讯云 CDN、Cloudflare、Akamai 等

## 13. TCP 和 UDP 的区别

| 特性   | TCP            | UDP           |
| ---- | -------------- | ------------- |
| 是否连接 | 面向连接（三次握手）     | 无连接           |
| 可靠性  | 可靠，顺序、重传、纠错    | 不可靠，可能丢包乱序    |
| 应用场景 | 文件传输、HTTP、数据库等 | 视频直播、游戏、DNS 等 |
| 速度   | 慢（要确认）         | 快（小包高频）       |

## 14. 会议系统会用什么协议？

* **UDP + RTP + SRTP** → 实时传输音视频数据（低延迟）
* **WebRTC**：浏览器实时通信协议，底层基于 UDP + ICE/STUN/TURN
* **信令通信**（非媒体）可用 TCP/HTTP/WebSocket


## 15. HTTP 和 HTTPS 的区别

* **HTTPS = HTTP + TLS/SSL 加密层**
* HTTP 是明文传输，HTTPS 是加密传输
* HTTPS 需要数字证书进行身份验证
* HTTPS 更安全，防劫持、防篡改、防监听

## 16. 数字证书的主要参数

* **公钥（Public Key）**
* **颁发者（Issuer）CA 签名者信息**
* **使用者（Subject）你是谁**
* **有效期（Valid From / To）**
* **证书序列号、签名算法**
* 可通过 `openssl` 或浏览器查看证书详情


## 17. 直播协议了解吗？

| 协议         | 特点                | 应用场景       |
| ---------- | ----------------- | ---------- |
| **RTMP**   | 基于 TCP，延迟低，传输稳定   | 主流推流协议     |
| **HLS**    | 基于 HTTP，切片传输，延迟高  | 点播、直播回看    |
| **FLV**    | HTTP-FLV，延迟较低，兼容好 | 网页直播       |
| **SRT**    | 新一代低延迟直播传输协议，抗丢包强 | 专业直播、大厂部署  |
| **WebRTC** | 端到端实时通信，超低延迟      | 互动直播、音视频通话 |

## 18. HashMap 和 Hashtable 区别？

* HashMap **线程不安全**，效率高；Hashtable **线程安全**（方法加锁），但性能差；
* HashMap 允许 `null` 键和值，Hashtable 不允许。

## 19. 针对 Hashtable 的问题，有什么更好方案？

* 使用 **ConcurrentHashMap** 替代：分段锁/锁分离/红黑树优化，**性能远超 Hashtable**
* JDK8 以后用 CAS + synchronized 替代 segment 更高效。

## 20. 线程和进程的区别？

* 线程是进程的执行单元，共享内存；进程是资源分配的最小单位，独立地址空间。
* **线程轻量、切换快；进程隔离好、稳定性高。**

## 21. 通信方式上线程与进程有啥区别？

* 线程间通信可直接读写共享变量（需加锁）
* 进程间通信需用 **IPC机制**：管道、消息队列、共享内存、socket 等

## 22. MySQL 索引结构？

* InnoDB 使用 **B+树** 索引，主键为聚簇索引，其他是二级索引
* 索引底层是磁盘优化的平衡多叉树结构

## 23. B 树和 B+ 树的区别？

* B树数据存储在**所有节点**，B+树**只存在叶子节点，且有链表连接**
* B+树结构更适合磁盘顺序读写

## 24. 为什么 B+ 树更适合范围查询？

* 所有数据在**叶子节点有序链表**中，直接按链表遍历即可，无需回溯，**效率更高**

## 25. MySQL 四种事务隔离级别？

| 隔离级别                | 问题           |
| ------------------- | ------------ |
| Read Uncommitted    | 会出现脏读        |
| Read Committed      | 避免脏读，但有不可重复读 |
| Repeatable Read（默认） | 避免不可重复读，但有幻读 |
| Serializable        | 最严格，防幻读，性能差  |

## 26. MVCC 是什么？

* 多版本并发控制，基于**隐藏字段**（创建版本号 + 删除版本号）
* 快照读 + undo log，实现 **读不加锁、并发高性能**

## 27. 索引失效的场景？

* `LIKE '%xxx'` 左模糊
* 隐式类型转换、函数包裹字段
* 未走最左前缀、范围后字段失效
* `or` 条件某字段无索引


## 28. Redis 持久化方案？

* **RDB**：定时快照，重启恢复快
* **AOF**：记录操作日志，安全性高
* 支持 RDB+AOF 混合使用

## 29. 缓存雪崩 / 穿透 / 击穿

| 问题 | 描述              | 解决方案            |
| -- | --------------- | --------------- |
| 雪崩 | 大量 key 同时失效     | 随机过期时间 + 限流 +降级 |
| 穿透 | 查不到的数据反复打数据库    | 缓存空值 / 布隆过滤器    |
| 击穿 | 热点 key 失效瞬时并发过高 | 分布式锁 / 本地互斥重建缓存 |

## 30. TCP 三次握手 / 四次挥手？

* **三次握手**：SYN → SYN-ACK → ACK，确保双向通信可达
* **四次挥手**：FIN → ACK → FIN → ACK，连接双向关闭，确保可靠释放


## 31. Linux 的常用命令？

* `ls`、`cd`、`pwd`、`rm`、`cp`、`mv`
* `ps -aux`、`top`、`grep`、`find`、`tail -f`
* `chmod`、`chown`、`netstat`、`curl`、`ping`

## 32. 用过 Docker 吗？常用命令？

* 常用命令：

  * `docker build` 构建镜像
  * `docker run` 启动容器
  * `docker ps` 查看容器
  * `docker exec` 进容器
  * `docker logs` 查看日志

* 构建镜像：写 `Dockerfile`，定义基础镜像、拷贝文件、启动命令等
  示例：

  ```Dockerfile
  FROM openjdk:8
  COPY app.jar /app.jar
  CMD ["java", "-jar", "/app.jar"]
  ```


## 33. 怎么解决消息重传问题？

> **答法：基于消息唯一标识 + 幂等性处理**

* MQ（如 Kafka、RocketMQ）可能因网络或消费失败重投
* 解决：**生产消息加唯一 ID（如订单号、UUID）**，消费端做去重

  * 可结合 Redis/数据库记录已处理 ID
  * 消费端实现**幂等消费逻辑**，避免重复处理同一消息

## 34. 怎么做消息幂等性处理？

> **核心：业务操作具备“重复执行效果相同”的能力**

* 给每条消息添加 **唯一业务 ID（如订单号、交易号）**
* 消费前先判断该 ID 是否已处理（查 Redis/数据库日志表）
* 如果处理过则跳过，没处理就执行操作并记录处理状态

## 35. Redis 的 key 已写入，但消费者宕机了怎么办？

> **答法：引入“消费确认机制 + 异步回查”**

* 若使用 Redis 作为幂等标记：

  * 消息 key 已写，但业务未执行，系统不一致
* **解决方案：**

  * **消息重试 + 幂等处理**（确保业务可重入）
  * 结合**消息事务机制**或中间件支持的事务消息（如 RocketMQ 事务消息）
  * 使用**可靠事件队列 + 补偿机制**

## 36. 建立索引的 SQL 语句？

```sql
-- 单列索引
CREATE INDEX idx_name ON table_name (column_name);

-- 联合索引（多个字段）
CREATE INDEX idx_name ON table_name (col1, col2);

-- 唯一索引
CREATE UNIQUE INDEX idx_name ON table_name (column_name);
```

## 37. TLS 握手过程（含证书验证和密钥协商）

> **答法：HTTPS加密通信底层就是 TLS**

1. **Client Hello**：客户端发起请求，附带支持的加密套件 + 随机数
2. **Server Hello**：服务端返回随机数 + 服务器数字证书（含公钥）
3. **客户端验证证书**：是否可信、合法、过期
4. **生成对称密钥**：

   * 客户端用服务器公钥加密对称密钥发送给服务器
   * 服务端用私钥解密获取该密钥
5. **握手完成，切换对称加密传输**（如 AES）

## 38. 对 student 表的 name 建索引：

```sql
CREATE INDEX idx_name ON student(name);
```


## 39. 主键索引 vs 普通索引

* 主键索引是**聚簇索引**，数据按主键顺序存储
* 普通索引是**非聚簇索引**，只存键值和主键指针

## 40. 非聚簇索引为什么要回表？

* 非聚簇索引存的不是完整数据，而是**主键指针**，需要回主键索引取整行数据


## 41. MySQL 四种隔离级别

* RU：脏读
* RC：不可重复读
* RR（默认）：幻读
* Serializable：最安全但效率最低


## 42. MySQL 主从同步原理

* 主库将写操作记录进 binlog → 从库 IO 线程读取 → SQL 线程回放执行 → 实现同步


## 43. binlog 存的是什么？

* MySQL 所有更改数据的操作（Insert/Update/Delete）的**二进制日志**


## 44. Redis 缓存雪崩是什么？怎么解决？

* 多个 key 同时过期导致大量请求打到数据库，可能造成宕机
* **解决：** 加随机过期时间、构建多级缓存、限流降级


## 45. Redis 持久化方式有哪些？

* RDB：快照持久化
* AOF：记录写操作日志
* 混合持久化：兼顾两者优势


## 46. HTTP 报文格式

* 请求报文 = 请求行 + 请求头 + 空行 + 请求体
* 响应报文 = 状态行 + 响应头 + 空行 + 响应体

## 47. HTTP 和 HTTPS 的区别

* HTTPS 多了 TLS 加密，数据更安全
* 需要证书认证和握手过程

## 48. TLS 握手过程简述

1. Client Hello（支持的加密算法+随机数）
2. Server Hello（证书+随机数）
3. 客户端验证证书并生成对称密钥
4. 客户端用公钥加密密钥发送给服务端
5. 开始对称加密通信


## 49. 为什么要结合对称加密和非对称加密？

* 非对称加密安全但慢，只用来传输对称密钥
* 对称加密快，用于大数据传输
* 两者结合兼顾安全与效率

## 50. Java 面向对象三大特性

封装、继承、多态

## 51. xxx.java 是如何变成二进制文件运行的？

* 先用编译器编译成 `.class` 字节码文件
* JVM 类加载器加载该文件
* 解释或 JIT 编译成机器码运行


## 52. JVM 的作用是什么？

* 跨平台运行 Java 字节码
* 管理内存（GC）、类加载、执行引擎、安全等


## 53. 线程和进程的区别

* 线程是轻量级的，多个线程共享进程资源
* 进程之间资源隔离，稳定性好但开销大

## 54. 协程和线程的区别

* 协程是用户态调度，不依赖系统线程调度器
* 更轻量，适合 I/O 密集型场景


## 55. 为什么协程更轻量？

* 切换只在用户态完成，无系统调用开销
* 占用资源更少，可同时运行成千上万协程


## 56. 二进制文件加载进内存后的结构？

* 分为：代码段、数据段、堆、栈、BSS 段
* ELF 文件还有节区表、符号表、程序头等结构


## 57. 如何查看系统负载？

```bash
uptime  # 查看 load average
```


## 58. top 命令哪些参数表示负载？

* 第一行的 `load average`
* `%CPU` 表示 CPU 占用情况


## 59. Linux 如何查看 CPU 核数？

```bash
lscpu | grep 'CPU(s)'
cat /proc/cpuinfo | grep "processor" | wc -l
```

## 60. 查看当前目录空间占用

```bash
du -sh ./
```


## 61. 查看当前服务器 TCP 连接数

```bash
netstat -an | grep ESTABLISHED
```

## 62. TCP 滑动窗口机制是什么？

* 控制未确认数据量的窗口机制，保证数据有序、流量受控、吞吐高


## 63. TIME\_WAIT 状态含义

* 主动断开连接的一方需要等待 2 倍 RTT，防止旧连接影响新连接

## 64. 为什么是四次挥手？

* TCP 是全双工，双方都要发送一次 FIN 和 ACK，才能完全断开连接


## 65. 三次握手中 ACK 和 SYN 是怎么合并的？

* 第二次握手由服务端发 `SYN + ACK` 一起，合并为一个报文段，节省一次传输


## 66. 容器之间是如何隔离的？

* 使用 Linux 的 Namespace（命名空间）和 CGroup（资源配额）机制
* 实现网络、进程、文件系统等资源隔离


## 67. Docker 镜像构建流程

1. 编写 Dockerfile（FROM、COPY、RUN、CMD 等）
2. `docker build -t xxx .` 构建镜像
3. `docker run` 运行容器


## 68. 大模型训练流程简述

数据准备 → 模型初始化 → 前向传播 → 反向传播（BP）→ 梯度更新 → 多轮训练 → 保存模型


## 69. MCP 是什么？

常见解释有：

* Memory Consistency Protocol：多核内存一致性协议
* Multi-Channel Processing：多通道处理架构

## 70. 常见 AI 工具有哪些？

* ChatGPT、GitHub Copilot、HuggingFace、Midjourney、Stable Diffusion、LangChain、PaddlePaddle 等

## 71. ArrayList 用过吗？是线程安全的吗？

用过，**不是线程安全的**，多个线程同时修改会出现数据不一致或异常。


## 72. 怎么保证 ArrayList 的线程安全？

常见方式有三种：

1. **使用 Collections.synchronizedList** 包装：

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

2. 使用 **CopyOnWriteArrayList**：适合读多写少场景

3. 手动加锁（synchronized 或 ReentrantLock）


## 73. 死锁问题怎么解决？

避免死锁的常用方法：

* **资源加锁顺序一致**
* **减少锁的持有时间**
* **加锁时使用 tryLock 设置超时**
* 检测与恢复：使用工具（如 jconsole）定位死锁线程


## 74. 进程和线程的区别？

| 对比项 | 进程            | 线程              |
| --- | ------------- | --------------- |
| 内存  | 拥有独立内存空间      | 共享进程内存          |
| 开销  | 创建、切换开销大      | 开销小             |
| 稳定性 | 一个崩溃不影响其他进程   | 一个崩溃可能影响进程中其他线程 |
| 通信  | 需要 IPC（进程间通信） | 共享变量即可通信        |


## 75. 线程共享的有什么？

* 同一进程中的线程共享：**内存地址空间、堆内存、方法区、静态变量、文件描述符等**
* 不共享的有：**线程栈（局部变量）、程序计数器、虚拟机栈**


## 76. JVM 怎么进行垃圾回收的？

JVM 垃圾回收依赖**GC算法 + 分代思想**：

* **新生代**：使用 **复制算法**（如 Minor GC）
* **老年代**：使用 **标记-清除/标记-整理**（如 Major GC）
* 常用 GC 收集器：**G1、ZGC、CMS、Serial、Parallel**

判断对象是否可回收：

* **引用计数法（已废弃）**
* **可达性分析（主流）**


## 77. MySQL 锁的分类？

按照粒度分类：

* **表级锁**：如 MyISAM 的锁，开销小，冲突多
* **行级锁**：如 InnoDB 的锁，开销大，冲突少

按照类型分类：

* **共享锁（S 锁）**：允许读，不允许写
* **排他锁（X 锁）**：允许自己读写，其他线程不能读写
* **意向锁**：表级锁，标记事务是否打算加行锁（加快锁冲突检测）
* **乐观锁/悲观锁**（逻辑层面的控制手段）

## 78. Redis - MySQL 一致性方案

缓存一致性常用方案：

* **先更新数据库，再删缓存**（推荐）
* **延迟双删机制**：先删缓存 → 更新数据库 → 延迟再删一次缓存
* 加**分布式锁**、**消息队列补偿机制** 处理并发/失败情况


## 79. MySQL 慢查询优化怎么做？

* 添加/优化**索引**
* 使用 **EXPLAIN** 查看执行计划
* 避免 \*\*select \*\*\*，减少扫描字段
* 加 limit，分页时用 **索引字段过滤**
* 拆大表、归档历史数据
* 设置慢查询日志排查：

  ```sql
  SET global slow_query_log = 1;
  ```

## 80. 索引失效场景有哪些？

* 使用函数或运算：

  ```sql
  WHERE YEAR(create_time) = 2024
  ```
* 字段类型不一致（隐式转换）
* 模糊查询左模糊：

  ```sql
  WHERE name LIKE '%abc'
  ```
* 多列索引不满足最左匹配原则
* 使用 or 且字段未都建索引


## 81. TCP 三次握手过程

1. 客户端发送 SYN 报文
2. 服务端返回 SYN + ACK
3. 客户端回复 ACK，连接建立完成

**为什么三次？**

* 保证双方都收到了彼此的能力确认（发送/接收）

## 82. TCP 拥塞控制机制有哪些？

1. **慢启动**（初始小窗口，指数增长）
2. **拥塞避免**（线性增长）
3. **快速重传**（三次 dup-ack 立即重传）
4. **快速恢复**（调整拥塞窗口）


## 83. CDN 业务逻辑简述

CDN（内容分发网络）主要作用：

* 将静态内容（图片/视频）**缓存到离用户更近的边缘节点**
* 加速访问、减轻源站压力、提高高并发吞吐
* 基本流程：DNS调度 → 找最近节点 → 命中缓存或回源

## 84. 算法：三数之和（Leetcode 15）

**题意：** 找到数组中三元组满足 a + b + c = 0，不能重复
**解法：** 排序 + 双指针

```python
def threeSum(nums):
    nums.sort()
    res = []
    for i in range(len(nums)-2):
        if i > 0 and nums[i] == nums[i-1]: continue
        l, r = i+1, len(nums)-1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s < 0: l += 1
            elif s > 0: r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                while l < r and nums[l] == nums[l+1]: l += 1
                while l < r and nums[r] == nums[r-1]: r -= 1
                l += 1; r -= 1
    return res
```


## 85. 算法：有序数组去重（快慢指针）

**题意：** 原地删除重复元素，返回去重后长度
**解法：** 快慢指针（双指针）

```python
def removeDuplicates(nums):
    if not nums: return 0
    slow = 0
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]
    return slow + 1
```

## 86. 算法：LCA 最近公共祖先（递归 + 哈希解法）

**题意：**

给定一棵二叉树和两个节点 `p` 和 `q`，找到它们的最近公共祖先。

**方法一：递归法（自顶向下）**

```python
def lowestCommonAncestor(root, p, q):
    if not root or root == p or root == q:
        return root
    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)
    if left and right: return root
    return left if left else right
```

**方法二：哈希 + 父指针**

1. 先用 DFS 构建 `node -> parent` 映射
2. 找出 p 的所有祖先存进 set
3. 从 q 向上找，遇到第一个在 set 中的即为 LCA

```python
def lowestCommonAncestor(root, p, q):
    parent = {}
    def dfs(node):
        if node.left:
            parent[node.left] = node
            dfs(node.left)
        if node.right:
            parent[node.right] = node
            dfs(node.right)
    dfs(root)

    ancestors = set()
    while p:
        ancestors.add(p)
        p = parent.get(p)
    while q not in ancestors:
        q = parent.get(q)
    return q
```


## 87. 图的搜索问题：DFS / BFS 应用场景

**DFS（深度优先搜索）：**

* 递归或栈实现
* **适合找路径、连通块、回溯、拓扑排序**

**示例：二维地图的岛屿数量**

```python
def numIslands(grid):
    def dfs(i, j):
        if i<0 or i>=len(grid) or j<0 or j>=len(grid[0]) or grid[i][j] != '1':
            return
        grid[i][j] = '0'
        for x, y in [(0,1),(1,0),(-1,0),(0,-1)]:
            dfs(i+x, j+y)

    count = 0
    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1':
                dfs(i, j)
                count += 1
    return count
```


**BFS（广度优先搜索）：**

* 使用队列，**适合最短路径、分层搜索、拓扑排序**

**示例：无权图最短路径**

```python
from collections import deque
def bfs_shortest_path(graph, start, end):
    visited = set()
    queue = deque([(start, [start])])
    while queue:
        node, path = queue.popleft()
        if node == end:
            return path
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                queue.append((neighbor, path + [neighbor]))
    return []
```

## 88. 手撕反转链表 — 迭代和递归实现

**迭代法**

思路：用两个指针 `prev` 和 `curr`，遍历链表，将 `curr.next` 指向 `prev`，逐步反转。

```python
def reverseList(head):
    prev = None
    curr = head
    while curr:
        next_node = curr.next  # 保存下一个节点
        curr.next = prev       # 反转指针
        prev = curr            # prev 后移
        curr = next_node       # curr 后移
    return prev  # prev 是新头节点
```



**递归法**

思路：递归到底后开始反转指针，最后返回新的头节点。

```python
def reverseList(head):
    if not head or not head.next:
        return head
    new_head = reverseList(head.next)
    head.next.next = head  # 让后一个节点指向当前节点
    head.next = None       # 当前节点断开后续连接
    return new_head
```

## 89. 手撕 单链表删除指定节点（无头指针，直接给定该节点）

**题意**

链表如 `1 → 2 → 3 → 4`，给定节点指针指向 `2`，要删除它，但没有头指针。

**关键点**

无法从头遍历，所以不能用常规删除。只能把后继节点数据复制到当前节点，再删掉后继节点。

**代码实现**

```python
def deleteNode(node):
    # node 不是尾节点
    node.val = node.next.val          # 复制后继节点值到当前节点
    node.next = node.next.next        # 删除后继节点
```

## 90. 手撕 LeetCode 199. 二叉树的右视图

**题意**

返回二叉树从右侧看到的节点值序列。

**思路**

层序遍历（BFS），每层取最后一个节点。

**代码实现**

```python
from collections import deque

def rightSideView(root):
    if not root:
        return []
    res = []
    queue = deque([root])
    while queue:
        size = len(queue)
        for i in range(size):
            node = queue.popleft()
            # 每层最后一个节点加入结果
            if i == size - 1:
                res.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
    return res
```

## 91. 手撕题：二叉树层序遍历（Null 用 `*` 占位）

**题意**

普通层序遍历基础上，遇到空节点用 `*` 代替，输出完整的层序结果。

**思路**

* 使用队列做层序遍历
* 遇到节点为空，加入 `*` 并继续占位
* 为防止无限扩展，遍历时判断当前层是否全为 `None`，如果是则停止

**代码实现（Python）**

```python
from collections import deque

def levelOrderWithNull(root):
    if not root:
        return []

    res = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        level_vals = []
        all_none = True  # 记录本层是否全是 None

        for _ in range(level_size):
            node = queue.popleft()
            if node:
                level_vals.append(node.val)
                queue.append(node.left)
                queue.append(node.right)
                if node.left or node.right:
                    all_none = False
            else:
                level_vals.append('*')
                queue.append(None)
                queue.append(None)

        # 如果本层全是 None，说明后面都是空，停止遍历
        if all_none and all(v == '*' for v in level_vals):
            break

        res.extend(level_vals)

    # 剔除末尾多余的 '*'
    while res and res[-1] == '*':
        res.pop()

    return res
```


## 92. 加载一个 HTML 的浏览器是怎么渲染页面的？

1. **解析 HTML**，构建 **DOM 树**
2. **解析 CSS**，构建 **CSSOM 树**
3. **合成渲染树（Render Tree）**，结合 DOM 和 CSSOM
4. **布局（Reflow）**，计算每个节点大小和位置
5. **绘制（Repaint）**，将节点绘制到屏幕上
6. 加载和执行 JS 会阻塞渲染，可能引发重排重绘


## 93. 重排和重绘在什么阶段做的？怎么尽量避免重排重绘？

* **重排（Reflow）**：DOM 结构或样式改变导致布局改变时发生，成本大
* **重绘（Repaint）**：只影响颜色、visibility 等视觉样式改变时发生，成本较小

**避免方法：**

* 减少频繁修改布局相关属性（如 width、height、margin）
* 批量修改样式，避免逐条修改触发多次重排
* 使用 `transform`、`opacity` 做动画，避免触发布局变更
* 使用文档碎片或隐藏元素批量操作 DOM


## 94. 浏览器 DOM 树和 Vue 里的虚拟 DOM 树关系？

* **浏览器 DOM 树**：真实的、由浏览器解析 HTML 生成
* **虚拟 DOM**：Vue 用 JS 对象描述的 DOM 树的轻量副本，存于内存
* Vue 修改虚拟 DOM 后，经过 Diff 算法计算出差异，再最小化操作更新真实 DOM，提升性能


## 95. 虚拟 DOM 树节点包含什么信息？

* 标签名（tag）
* 属性（props）和样式
* 子节点（children）
* 唯一 key（用于优化 diff）
* 组件状态等元数据


## 96. 往列表插入元素，虚拟 DOM 树怎么处理？

* 新元素对应的虚拟节点插入到对应位置
* Diff 算法比较新旧虚拟 DOM，找出最小差异
* 只对比变动节点，执行对应的增删改操作更新真实 DOM


## 97. Diff 算法怎么操作的？

* 逐层比较新旧虚拟 DOM
* 利用 **key** 优化节点匹配
* 通过最小编辑距离策略，尽量减少节点移动和删除重建
* 保持子树顺序，避免不必要的 DOM 操作，提高性能


## 98. CSS 中的 BFC（块级格式上下文）？

* BFC 是一种独立的渲染区域，内部元素布局互不影响外部
* 触发条件：`overflow` 非 `visible`、`display: flow-root`、浮动元素、绝对定位等
* BFC 作用：清除内部浮动、避免外边距重叠、限制元素的盒子范围
* 是理解布局和避免布局问题的重要概念


## 99. JS 中的闭包（闭包是什么？原理？）

* 闭包是函数和声明该函数的词法环境组合的结构
* 函数能访问其外部作用域变量，即使外部函数执行完毕
* 原理：JS 函数在创建时会保存作用域链，形成闭包
* 常用于封装私有变量、实现模块化和回调函数


## 100. 怎么用闭包实现单例设计模式？

```js
const Singleton = (function() {
  let instance;
  function createInstance() {
    return { /* 实例对象 */ };
  }
  return function() {
    if (!instance) {
      instance = createInstance();
    }
    return instance;
  }
})();
```

调用 `Singleton()` 始终返回同一个实例。

## 101. JS 怎么改变函数作用域？

* 使用 **call/apply/bind** 改变函数执行时的 `this` 指向
* 使用 **with**（不推荐）
* 利用 **eval** 动态执行代码
* 利用闭包包裹不同作用域变量
* ES6 的箭头函数绑定定义时的 `this`，不受调用时影响

## 102. 算法题：最长无重复字符子串（滑动窗口）

**题意**

给定一个字符串，找出其中最长的没有重复字符的子串长度。

**思路**

用滑动窗口 + 哈希表记录字符最后出现的位置，窗口内保持无重复。

**代码实现（Python）**

```python
def lengthOfLongestSubstring(s):
    char_index = {}
    left = 0
    max_len = 0

    for right, char in enumerate(s):
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1  # 移动左指针，跳过重复字符
        char_index[char] = right
        max_len = max(max_len, right - left + 1)

    return max_len
```

**说明**

* `left` 和 `right` 是滑动窗口左右边界
* `char_index` 存储字符最近出现的位置
* 复杂度 O(n)，适合面试直接写











