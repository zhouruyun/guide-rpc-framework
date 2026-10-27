# 手写 RPC 框架 · guide-rpc-framework

一个从零实现的轻量级 RPC 框架，用来理解「一次远程调用背后到底发生了什么」：协议设计、序列化、网络通信、服务注册发现、负载均衡、动态代理。

> 本项目跟随开源教程 [Snailclimb/guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework) 实现，用于学习 Netty 与 RPC 原理。

![Java](https://img.shields.io/badge/Java-8+-ED8B00?logo=openjdk&logoColor=white)
![Netty](https://img.shields.io/badge/Netty-4.x-2E7D32)
![Zookeeper](https://img.shields.io/badge/Zookeeper-3.7-FF6A00)

## 特性

- **自定义通信协议**：魔数 + 版本号 + 序列化类型 + 消息长度 + 消息体，解决 TCP 粘包 / 拆包
- **多种序列化**：Kryo、Protostuff、JDK 原生，通过 SPI 扩展，报文体积较 JDK 序列化显著减小
- **服务注册与发现**：Zookeeper 临时节点 + 监听，支持服务上下线感知与客户端自动重连
- **负载均衡**：随机、轮询、一致性哈希三种策略
- **透明调用**：JDK 动态代理屏蔽远程调用细节，调用方像用本地方法一样
- **异步调用**：基于 `CompletableFuture` 支持异步获取结果
- **容错**：失败重试、心跳检测、空闲连接关闭

## 模块结构

```
guide-rpc-framework
├── rpc-framework-common   # 协议、编解码、序列化、SPI、异常
├── rpc-framework-simple   # Netty 服务端 / 客户端、注册中心、负载均衡、代理
├── hello-service-api      # 示例服务接口（被调用方与调用方共享）
├── example-server         # 服务提供者
├── example-client         # 服务消费者
└── docs                   # 设计笔记
```

## 运行

```bash
# 1. 启动 Zookeeper
docker run -d --name zk -p 2181:2181 zookeeper:3.7

# 2. 启动服务端
mvn -pl example-server spring-boot:run

# 3. 启动客户端，观察调用结果
mvn -pl example-client spring-boot:run
```

## 关键流程

```
客户端调用接口
  → 动态代理生成代理对象
  → 组装 RpcRequest（接口名、方法名、参数、参数类型）
  → 序列化 + 按自定义协议编码
  → Netty 发送 → 服务端解码 → 反射调用本地实现
  → 封装 RpcResponse 回写 → 客户端解码 → 返回结果
```

## 待补充

- [ ] 补充 JMeter 压测数据（QPS / 平均 RT）
- [ ] 增加服务端限流与线程池隔离
- [ ] 补充注册中心（Nacos）实现，对比 Zookeeper 的差异

## 致谢

- 原项目：[Snailclimb/guide-rpc-framework](https://github.com/Snailclimb/guide-rpc-framework)
- 教程：《手把手教你写一个 RPC 框架》
