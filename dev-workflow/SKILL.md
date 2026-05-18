---
name: dev-workflow
description: 当用户提出代码修改、功能开发、bug修复、架构设计、需求讨论、项目规划或任何软件开发相关请求时调用 — 智能路由变更到对应规模流程
version: 3.1.0
---

# Dev Workflow

> 核心理念: 用户聊天。AI 感知阶段、创建文档、追踪关联、发现冲突。
> 详细规则 → `REFERENCE.md`。此文件保持 < 120 行，超过即拆分。

## 路由决策

每次加载后，根据变更信号判断规模并选择路径：

| 规模 | 判断 | 路径 |
|------|------|------|
| **微小** | ≤1 文件、无设计选择、意图明确 | 直接执行，可选记入 CHANGES.md |
| **小型** | ≤3 文件、无架构决策、选择清晰 | 快速追问(≤3问) → SPEC → 执行 |
| **中型** | 多文件/模块、有设计取舍 | BRIEF → ADR → SPEC → PHASE |
| **大型** | 跨系统、架构影响、破坏性变更 | 完整仪式 + 收敛扫描 + 里程碑 |

确定路由后告知用户，用户可要求升降级。纯信息询问不进入任何路径。→ `REFERENCE.md §28`

显式触发：`/dev-workflow`。

## 阶段感知与流转（中大型适用）

| 信号 | 阶段 | 动作 |
|------|------|------|
| "我想做..."、描述场景痛点 | **头脑风暴** | 创建 BRIEF，追问 ≤5 问，更新 DESIGN-BOARD |
| "怎么做""选哪个" | **澄清** | 提炼决策点，建议创建架构文档 |
| 澄清完成、方案待验证 | **拷问** | 主动遍历决策树，一次一问，消除歧义 |
| 二选一/多选一 | **固化** | 起草 ADR，列利弊，请确认 |
| "开干""做吧" | **施工** | 创建 SPEC + PHASE，按规格执行 |
| 施工完成/要求审查 | **验证** | 审查匹配 SPEC，更新 PHASE |
| 新想法 vs 旧决策冲突 | **收敛** | 调 conflict-resolver，标记影响范围 |

**优先级**: 冲突 > 收敛 > 拷问 > 澄清 > 施工 > 头脑风暴。用户说"先做着"→ 施工优先。
**多想法**: 独立方向各建 BRIEF；同方向子项放一个 BRIEF。→ `REFERENCE.md §1.3`

## 微型/小型路径

**微小**: 确认范围 → 直接执行 → 有 `docs/` 时追加 `docs/CHANGES.md`。不做 BRIEF/ADR/SPEC/PHASE，不触发扫描。
**小型**: 快速追问(≤3问) → 简化 SPEC(跳过模块分析) → 施工 + 标记完成。不经过 ADR，不触发收敛扫描。
→ `REFERENCE.md §29-30`

## PR 自动化

| 步骤 | 触发 | 动作 |
|------|------|------|
| 创建 | PHASE 完成 | `gh pr create` + 关联 SPEC |
| 合并 | PR 已合并 | 执行 `post-merge.sh`（若存在） |
| 发布 | 所有 SPEC 完成 | 更新 MILESTONE + `git tag` |

钩子 `.claude/workflow-hooks/post-merge.sh` 由用户预写，AI 仅执行。无钩子时仅更新状态 + git tag。→ `REFERENCE.md §31`

## 核心规则

1. **渐进落笔**: 微小→直接执行; 小型→SPEC(可执行); 中型→BRIEF→ADR→SPEC; 大型→完整仪式+里程碑
2. **冲突即对话**: 静默扫描 → 分类（轻微/根本）→ 列出影响范围 → 问用户。施工中遇冲突 → PAUSE PHASE。→ `REFERENCE.md §4`
3. **收敛自动**: 每 3 ADR / 5 SPEC → converge-scanner 扫描。→ `REFERENCE.md §5`
4. **会话恢复**: 新会话读 DESIGN-BOARD → 活跃 PHASE → 暂停 PHASE → 未读 REVIEW。→ `REFERENCE.md §6`
5. **看板一句话**: 用户说"看板" → 只读 DESIGN-BOARD + 活跃 PHASE + 未读 REVIEW，结构化呈现。→ `REFERENCE.md §20`
6. **并行可控**: 独立 SPEC 可同时施工，AI 检测文件重叠。→ `REFERENCE.md §23`
7. **ADR 链式**: 每个 ADR 声明依赖和被依赖。废弃时级联通知。→ `REFERENCE.md §24`
8. **实验不丢**: 改方向时 `git branch` 保存旧代码，不删除。→ `REFERENCE.md §22`
9. **拷问驱动**: 澄清后不直接固化——主动遍历决策树各分支，一次只问一个问题，压迫边缘情况。ADR 仅在三条件同时满足时创建（难以逆转+缺少上下文会令人惊讶+是真实权衡），否则记入 SPEC 备注。→ `REFERENCE.md §32, §35`
10. **垂直切片**: SPEC 拆 PHASE 优先端到端垂直切片（贯穿所有层、独立可演示），避免水平切片（按层/按文件类型拆分）。→ `REFERENCE.md §34`
11. **领域语言**: 项目术语定义于 `docs/GLOSSARY.md`。所有文档引用一致术语。拷问阶段的核心输出之一是消除术语歧义。→ `REFERENCE.md §33`

## 文件体系

在项目根创建 `docs/`（插件项目与 `.claude-plugin/` 同级）：

```
docs/
├── DESIGN-BOARD.md          # 持续更新
├── CONSTITUTION.md           # 几乎不变（含 NFR）
├── CHANGES.md                # 微小变更日志（AI 自动维护）
├── GLOSSARY.md               # 项目领域术语表（消除歧义）
├── ARCHITECTURE.md           # 偶尔变
├── briefs/   BRIEF-NNN-slug.md
├── adr/      ADR-NNN-slug.md
├── specs/    SPEC-NNN-slug.md
├── phases/   PHASE-NNN-slug.md
├── milestones/ MILESTONE-NNN-vX.Y.Z.md
└── reviews/  REVIEW-YYYY-MM-DD.md
```

既有项目：代码倒推 DESIGN-BOARD（7 步），单体文档解构+并行+渐进迁移。→ `REFERENCE.md §7`

## 命名规范

`{TYPE}-{NNN}-{slug}.md`，编号自动递增。slug 用小写连字符。与已有 DDR 体系并存，沿用现有编号。

## Skill 自维护与拆分规则

### 本 Skill 的维护

1. **SKILL.md ≤ 120 行**。超过时，核心规则留在 SKILL.md，细节移入 REFERENCE.md
2. **每次修改后检查行数**。AI 主动提醒："当前 SKILL.md 已 N 行，建议移 X 到 REFERENCE.md"
3. **模板和 agent 独立文件**，不计入行数

### 用本 Skill 构建其他 Skill/文件时

4. **任何 SKILL.md > 200 行 → AI 必须建议拆分**
5. **拆分标准**: 核心行为规则（永远需要）→ SKILL.md；细节/边界/示例 → 独立文件
6. **非 Skill 的 Markdown 文件 > 200 行 → 主动提示用户考虑拆分**

## 参考

详细规则按需加载：`REFERENCE.md` — 包含变更规模判断、微型/小型路径、阶段细节（含拷问环节）、冲突管理、收敛扫描、既有项目改造、质量门禁、拒绝处理、范围管理、Git 集成、模板使用、错误恢复、PHASE 状态机、并行施工、ADR 依赖链、NFR、里程碑、PR 自动化、垂直切片、领域语言、使用示例
