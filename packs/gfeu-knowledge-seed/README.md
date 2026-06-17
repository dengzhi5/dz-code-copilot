# gfeu-knowledge — 部门共享知识库

gfeu-code-copilot 框架的**部门级知识库**。与框架仓库、业务项目相互独立，承载跨项目复用的通用知识（技术约定、踩坑、反模式）。

## 它和谁配合

三层知识体系，覆盖语义统一为 **项目级 > 部门级**：

| 层 | 位置 | 内容 |
|----|------|------|
| 项目级 | `<业务项目>/gfeu-code-copilot/knowledge/` | 业务规则、本项目特有约束 |
| **部门级（本库）** | `~/.claude/gfeu-knowledge/` | 通用技术约定、踩坑、反模式 |

`/archive` 沉淀知识时，AI 判定每条是 project 还是 dept，dept 级写入本库；`/propose` 的 Research 阶段会同时加载项目级与部门级索引。

## 安装（每位部门成员各做一次）

clone 到约定路径——框架提示词硬编码读取 `~/.claude/gfeu-knowledge/`，路径不可改：

```bash
git clone <部门知识库仓库地址> ~/.claude/gfeu-knowledge
```

未 clone 也不影响框架运行：`/propose` 会跳过部门级加载，`/archive` 会把 dept 级条目暂存为 project 级并提示你 clone。

## 目录结构

```
gfeu-knowledge/
├── index.md            # 知识索引（关键词 → 文件），AI 在 /archive 时维护
├── tech/               # 技术约定
├── pitfalls/           # 踩坑记录
└── anti-patterns/      # 反模式
```

## 知识怎么进库

1. 在业务项目里走完 `/apply` → `/review`，执行 `/archive`
2. AI 逐条展示知识并标注【建议分级】，dept 级条目写入本库 + 更新 `index.md`
3. 框架**不自动 push**。你手动提交并发 MR：

```bash
cd ~/.claude/gfeu-knowledge
git add . && git commit -m "knowledge: <一句话摘要>"
git push   # 或发 MR，由 owner 评审合并
```

## 维护

- MR 评审 = 知识入库门禁，避免低质量条目淹没库
- owner 定期 review `index.md` 关键词覆盖度——关键词写不准，知识加载不出来，飞轮就空转
- 成员定期 `git pull` 保持本地新鲜
