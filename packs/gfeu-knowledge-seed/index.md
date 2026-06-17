# 部门知识库索引

> 本文件由 AI 在 `/archive` 沉淀 **dept 级** 知识时自动维护。
> 格式规范：`- **关键词**: 一句话摘要 → \`目录/文件名.md\``
> 关键词用于按需加载：`/propose` Research 阶段命中关键词时，读取对应文档补充上下文。
>
> ⚠️ 关键词质量决定飞轮是否转得起来。写不准 → 知识沉进去也加载不出来。
> 由部门知识库 owner 定期 review 关键词覆盖度。

## 技术约定（tech/）

<!-- 示例：
- **分布式锁**: 统一用 RedisLockHelper，TTL 默认 30s → `tech/redis-lock.md`
- **分页查询**: 禁止 SELECT *，超过 1000 条必须分批 → `tech/query-rules.md`
-->

（暂无，通过 /archive 沉淀 dept 级条目）

## 踩坑记录（pitfalls/）

<!-- 示例：
- **事务嵌套**: @Transactional 同类内部调用不生效，必须注入自身或拆服务 → `pitfalls/transaction-pitfalls.md`
- **MyBatis 批量插入**: 超过 1000 条需分批，否则 OOM → `pitfalls/mybatis-batch-insert.md`
-->

（暂无，通过 /archive 沉淀 dept 级条目）

## 反模式（anti-patterns/）

<!-- 示例：
- **Consumer 同步调第三方**: MQ 线程阻塞导致消费堆积，必须异步化 → `anti-patterns/sync-call-in-consumer.md`
-->

（暂无，通过 /archive 沉淀 dept 级条目）
