# CodeGraph 集成设计文档

**日期：** 2026-06-08  
**状态：** 已确认，待实现

---

## 背景

CodeGraph 是一个本地语义代码图谱工具，通过 MCP server 向 Claude Code 暴露 `codegraph_explore`、`codegraph_search`、`codegraph_callers`、`codegraph_impact` 等工具。基准测试显示平均节省 ~16% cost、~47% tokens、~58% tool calls。

gfeu-code-copilot 的 `/propose` Research、`/brainstorm` 探索现状、调试 Phase 1 三个阶段高度依赖 grep/glob/Read 做代码探索，是 CodeGraph 的直接受益点。

---

## 方案选择

**选定方案 B：显式工具映射，内联在对应命令章节**

- 在每个探索阶段直接给出"首选工具 → 降级工具"映射
- CodeGraph 不存在时 graceful degradation 为现有 grep/read 流程
- 不新增独立章节，保持提示词结构不变

---

## 变更范围（三层）

### 层1：`agents/copilot-prompt.md` — 三处内联工具映射

**变更点1：`/propose` Step 1 Research**（第173行前）

在现有 3 条 bullet 前插入：

```
  ⚡ 工具优先级（若 CodeGraph MCP 可用）：
  - 入口类 / 核心链路   → codegraph_explore("<需求关键词>")
  - 符号查找           → codegraph_search("<类名/方法名>")
  - 影响范围评估       → codegraph_impact("<核心符号>")
  - 文件结构           → codegraph_files
  - 无 CodeGraph 时降级：grep / glob / Read
```

**变更点2：`/brainstorm` Step 2 探索现状**（第136行前）

```
  ⚡ 工具优先级（若 CodeGraph MCP 可用）：
  - 探索现有实现       → codegraph_explore("<相关模块/类名>")
  - 调用方查找         → codegraph_callers("<方法名>")
  - 无 CodeGraph 时降级：Read 相关代码文件
```

**变更点3：调试 Phase 1 根因调查**（第342行前）

```
  ⚡ 工具优先级（若 CodeGraph MCP 可用）：
  - 调用链追溯         → codegraph_callers("<报错方法>")
  - 影响范围           → codegraph_impact("<可疑符号>")
  - 无 CodeGraph 时降级：git diff HEAD~3 + grep
```

### 层2：`agents/copilot-prompt.md` — `/init` 第8步

在 `/init` 流程报告步之后添加可选提示步骤：

```
8. （可选）提示 CodeGraph 初始化：
   "检测到项目代码库，建议运行以下命令为本项目建立语义代码图谱，
   可大幅减少后续 Research 阶段的 token 消耗：

   codegraph init -i

   （需先全局安装：curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh）
   若已安装直接运行 codegraph init -i 即可。"
```

### 层3：`install.sh` — 完成提示新增可选增强段

在完成 banner 之后、"下一步"之前插入：

```bash
echo ""
echo "${BOLD}可选增强（推荐）:${RESET}"
echo "  安装 CodeGraph 可让 AI 用预建代码图谱代替 grep 探索，"
echo "  减少约 58% 工具调用、节省 ~16% token："
echo "  ${BOLD}curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh${RESET}"
echo "  安装后在每个项目里运行: ${BOLD}codegraph init -i${RESET}"
echo ""
```

---

## 设计约束

- **零强依赖**：CodeGraph 不存在时，框架行为与现在完全一致
- **不改流程结构**：三个命令的 Step 编号、HARD-GATE、文档产出均不变
- **仅提示一次**：CodeGraph 初始化建议只在 `/init` 完成时触发，不在每次 Research 前重复
- **不自动安装**：install.sh 只展示命令，不自动执行 CodeGraph 安装

---

## 不在本次范围内

- 自动检测 CodeGraph 是否已安装（运行时判断由 AI 自然感知 MCP 工具是否存在）
- 修改 hooks/session-start
- 修改 rules/ 或 packs/ 下的任何规则文件
- code-quality-reviewer.md / spec-reviewer.md 的修改
