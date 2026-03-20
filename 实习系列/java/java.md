# java



### 一、 java基础

#### 1. java的特点

- **平台无关性**：字节码可以在任何安装了 JVM 的系统上运行
- **面向对象**：Java是一门严格的面向对象编程语言，几乎一切都是对象
- **内存管理**：java有GC，不需要自己手动管理内存，减少内存泄漏问题

### 2. java的优势和劣势

- **生态系统成熟**
  - 中间件统治力： 在企业级开发中，Spring 全家桶（Spring Boot/Cloud）几乎制定了行业标准。无论是 ORM、RPC（Dubbo/gRPC）、消息队列（Kafka/RocketMQ 客户端），Java 都有最成熟、经过大规模生产验证的实现
  - 大数据领域： Hadoop, Spark, Flink, ElasticSearch，在大数据基础设施层，JVM 语言（Java/Scala）占据绝对统治地位

- **JVM**：cpp虽然快，但很依赖于开发者的水平；java通过JVM兜底
  - JIT：虽然 Java 是解释执行起步，但在热点代码被 JIT 编译成本地机器码后，利用内联优化 、逃逸分析等技术，其峰值性能在很多场景下可以持平甚至超越 C++
  - GC：从 CMS 到 G1，再到现在的 ZGC 和 Shenandoah。ZGC 实现了 TB 级堆内存下 <1ms 的停顿时间，这解决了 Java 长期以来的“Stop-The-World”痛点，让 Java 能胜任低延迟交易系统

- **JMM和并发**
  - 规范化和并发：Java 的 volatile、synchronized 关键字背后的 **Happens-Before 原则**，屏蔽了不同 CPU 架构（x86, ARM）的内存屏障差异
  - J.U.C 包： Doug Lea 大神写的 java.util.concurrent (AQS, CAS, ConcurrentHashMap) 提供了极高质量的并发工具，比许多语言的原始锁机制更高效、更安全

- **可观测性与排错**
  - 工具链：JVisualVM, JProfiler, Arthas
  - 字节码注入： 利用 Java Agent 技术，可以在不重启服务的情况下进行动态追踪（Trace）、热替换代码。这是 APM（应用性能监控）工具（如 SkyWalking, Pinpoint）的基础