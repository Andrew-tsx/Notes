# RedisManager 和 CacheCloud 简介

## 什么是 Redis？

Redis 是一款非常流行的**内存数据库**，常被用来做缓存——可以理解为给网站/App 装了一个"超级快的短期记忆"，让数据读取速度飞快。但当你有很多台服务器、很多个 Redis 实例时，手动管理就会很痛苦，于是就有了管理工具。

---

## RedisManager

**一句话概括**：Redis 的**一站式运维管理平台**（Web 端）。

### 主要功能

- 可视化监控 Redis 集群状态
- 支持创建和管理集群（Cluster、主从、哨兵模式）
- 故障转移、告警通知
- 数据的增删改查操作

### 开源地址

- [ngbdf/redis-manager](https://github.com/ngbdf/redis-manager)

### 适合场景

已经有 Redis 在跑了，需要一个 Web 界面来统一监控和管理它们。

---

## CacheCloud

**一句话概括**：由**搜狐视频团队开源**的 Redis 云管理平台，定位更偏"私有云/PaaS"。

### 主要功能

- 一键自动部署 Redis（单机、哨兵、集群三种架构）
- 弹性伸缩（在线扩容/缩容）
- 完善的统计监控和报警
- 客户端接入整合（简化开发接入）
- 解决大规模实例碎片化问题

### 开源地址

- [sohutv/cachecloud](https://github.com/sohutv/cachecloud)
- [sohutv/cachecloud-client](https://github.com/sohutv/cachecloud-client)（客户端项目）

### 适合场景

公司/团队有大量 Redis 实例，需要像"云平台"一样统一申请、部署、管理，适合中大型团队。

---

## 对比

| 维度 | RedisManager | CacheCloud |
|------|-------------|------------|
| 定位 | 运维管理平台 | Redis 私有云平台 |
| 核心优势 | 监控、告警、故障转移 | 自动部署、弹性伸缩、全生命周期管理 |
| 适合规模 | 中小规模 | 中大型（数百实例） |
| 来源 | 社区开源 | 搜狐视频团队开源 |

> 如果是新手，**RedisManager 上手更简单**；如果团队规模较大，**CacheCloud 功能更全面**。

---

## 参考资料

- [Redis运维利器——RedisManager - 博客园](https://www.cnblogs.com/felixzh/p/11170051.html)
- [RedisManager GitHub](https://github.com/ngbdf/redis-manager)
- [CacheCloud GitHub](https://github.com/sohutv/cachecloud)
- [Redis云管理平台CacheCloud - CSDN](https://blog.csdn.net/liuwei0376/article/details/105223002)
- [CacheCloud云平台介绍 - GitCode博客](https://blog.gitcode.com/4a841b95b18fdfc5ad2dacf73e15c461.html)
- [Redis管理工具比对 - 掘金](https://juejin.cn/post/7023320399630827551)
