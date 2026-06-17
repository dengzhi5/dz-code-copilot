# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 这是什么仓库

这不是一个应用代码仓库，而是 **gfeu-code-copilot 框架本身的源码**——一套面向后端项目的 Codex 编码协作框架（Spec 驱动 + 渐进式复杂度）。仓库内容是 Markdown 提示词、Sub-Agent 定义、规则文件和安装脚本，没有需要编译的业务代码。

编辑本仓库时，你修改的是**框架的行为**；它通过 `install.sh` 安装到 `~/.Codex/` 后，会作为 skill 注入到其他项目的 Codex 会话里运行。

## 安装与分发机制（理解这点才能正确改框架）

`install.sh` / `install.ps1` 做三件事，改动相关文件时要保持一致：

1. **clone/pull 到** `~/.Codex/gfeu-code-copilot/`（本仓库的部署位置）
2. **symlink** `~/.Codex/skills/gfeu-code-copilot` → `<install>/skill/`（注册 skill；Windows 用 Junction）
3. **注入 SessionStart hook** 到 `~/.Codex/settings.json`，每次会话运行 `hooks/session-start`

关键路径耦合：提示词里大量硬编码 `~/.Codex/gfeu-code-copilot/...` 绝对路径（如 `agents/copilot-prompt.md`、`packs/`、`rules/`）。**移动或重命名顶层目录会破坏这些引用**——改目录结构时必须同步 grep 全仓库的路径引用。

## 框架运行链路

```
SessionStart hook (hooks/session-start)
  └─ 注入 <gfeu-code-copilot-safety-rules> 安全规则（HARD-GATE + 安全红线）
skill/SKILL.md  (用户触发 skill 时加载)
  └─ 要求立即 Read agents/copilot-prompt.md
agents/copilot-prompt.md  (主提示词 / 全部命令逻辑)
  ├─ 意图识别 → 复杂度分档 (Quick/Standard/Complex)
  ├─ 命令: init/brainstorm/propose/apply/fix/review/test/archive
  └─ /review 时以独立上下文调用两个 Sub-Agent:
       agents/spec-reviewer.md         (阶段一: Spec 合规)
       agents/code-quality-reviewer.md (阶段二: 代码质量)
```

`agents/copilot-prompt.md` 是框架大脑——绝大多数行为改动都在这里。三个 `agents/*.md` 之间有契约关系：copilot-prompt 描述的命令流程必须和 reviewer 的输入/输出约定一致。

## 顶层目录职责

| 目录 | 作用 | 改动注意 |
|------|------|---------|
| `agents/` | 主提示词 + 两个 reviewer Sub-Agent | 改命令流程的主战场，注意三者契约一致 |
| `skill/SKILL.md` | Codex skill 注册入口 | description 决定何时触发；正文指向 copilot-prompt |
| `hooks/` | SessionStart hook 脚本 + 配置 | `session-start` 输出 JSON，安全规则在此硬编码 |
| `rules/` | 全局编码规范（项目级可同名覆盖） | code-quality-reviewer 读这里做审查标准 |
| `packs/` | 技术栈规则包（如 java-spring），`/init` 按文件存在性检测加载 | 含扫描/构建/测试命令模板 |
| `knowledge/` | 全局知识库，`/archive` 写入，`/propose` 按 index.md 关键词加载 | |
| `changes/templates/` | 变更文档模板（design-brief/spec/tasks/log） | `/propose` 等命令从这里填充 |
| `anti-patterns/` | 反模式知识条目 | |
| `docs/` | 框架设计文档（flow / overview） | |

## 全局层 vs 项目层（覆盖语义）

框架运行时遵循**项目级优先于全局级**：

- 全局：`~/.Codex/gfeu-code-copilot/rules|knowledge|...`（即本仓库部署后的位置）
- 项目：`<目标项目>/gfeu-code-copilot/rules|knowledge|changes/...`（用户 `/init` 生成）

同名规则文件，项目级覆盖全局。本仓库里的 `rules/` 是**全局默认模板**，不是某个具体项目的配置。

### 知识三层与部门共享库

知识有独立的三层体系（覆盖语义同样是「项目 > 部门」）：

- 项目级：`<项目>/gfeu-code-copilot/knowledge/` — 业务规则、本项目特有约束
- **部门级**：`~/.Codex/gfeu-knowledge/` — 跨项目复用的技术约定/踩坑/反模式，是一个**独立 git 仓库**，部门成员各自 clone 到此固定路径
- 全局模板：`~/.Codex/gfeu-code-copilot/knowledge/` — 仅模板

`/archive` 时 AI 判定每条知识是 project 还是 dept，分别写入项目库 / 部门库；`/propose` 的 Research 阶段同时加载项目级与部门级索引（关键词冲突时两条都展示，项目级优先）。部门库的种子模板见 `packs/gfeu-knowledge-seed/`（README + index 模板）。框架对部门库**不自动 push**——dept 级知识写入后提示用户手动 commit/MR。

## 修改框架时的硬性约束

源自框架自身的设计原则，改提示词时不要削弱：

- **HARD-GATE 不可绕过**：Standard/Complex 档 spec 未确认禁止编码；`/review` 检测不到 `/apply` 提交则拒绝执行。这些门控逻辑同时存在于 `hooks/session-start`、`skill/SKILL.md`、`agents/copilot-prompt.md`，改一处要三处对齐。
- **Evidence Before Claims**：apply/fix/test 完成必须展示真实命令输出，禁止无证据的"应该没问题"。
- **每个结论标注代码出处**（文件路径 + 类名/方法名），reviewer 必须亲自读代码而非信报告。
- 安全红线（硬编码密钥、敏感信息日志、资金/状态/权限变更）在 `hooks/session-start` 和 `copilot-prompt.md` 双份维护，改一处同步另一处。

## 验证改动

没有自动化测试。改完框架后用人工方式验证：

```bash
bash hooks/session-start        # 应输出合法 JSON（含 additionalContext）
bash install.sh                 # 在隔离环境验证安装/更新链路（会写 ~/.Codex/）
bash install.sh --uninstall     # 验证卸载
```

提示词改动的"测试"是：在真实目标项目里触发 skill，走一遍对应命令流程，看行为是否符合预期。

## 关于在本仓库内使用框架命令

注意区分两种身份：当你在**本仓库**工作时，是在**开发框架**，应直接编辑 Markdown，不必走 `/propose → /apply` 流程（那是给框架的使用者、在他们的业务项目里用的）。除非用户明确要求按框架自身流程开发框架。