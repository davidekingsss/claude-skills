---
name: dev-workflow
description: 当用户说"我想做""怎么设计""帮我规划""讨论架构""加新功能""重构"时调用 — 五阶段渐进式开发流程
version: 2.0.0
---

# Dev Workflow

> 核心理念: 用户聊天。AI 感知阶段、创建文档、追踪关联、发现冲突。
> 详细规则 → `REFERENCE.md`。此文件保持 < 120 行，超过即拆分。

## 激活

检测以下信号时自动加载：描述新项目、讨论架构、管理需求、既有项目改造。显式触发：`/dev-workflow`。

## 阶段感知与流转

| 信号 | 阶段 | 动作 |
|------|------|------|
| "我想做..."、描述场景痛点 | **头脑风暴** | 创建 BRIEF，追问 ≤5 问，更新 DESIGN-BOARD |
| "怎么做""选哪个" | **澄清** | 提炼决策点，建议创建架构文档 |
| 二选一/多选一 | **固化** | 起草 ADR，列利弊，请确认 |
| "开干""做吧" | **施工** | 创建 SPEC + PHASE，按规格执行 |
| 施工完成/要求审查 | **验证** | 审查匹配 SPEC，更新 PHASE |
| 新想法 vs 旧决策冲突 | **收敛** | 调 conflict-resolver，标记影响范围 |

**优先级**: 冲突 > 收敛 > 澄清 > 施工 > 头脑风暴。用户说"先做着"→ 施工优先。

**多想法**: 独立方向各建 BRIEF；同方向子项放一个 BRIEF。→ `REFERENCE.md §1.3`

## 核心规则

1. **渐进落笔**: 想法 → BRIEF（模糊）→ ADR（精确）→ SPEC（可执行）
2. **冲突即对话**: 静默扫描 → 分类（轻微/根本）→ 列出影响范围 → 问用户。施工中遇冲突 → PAUSE PHASE。→ `REFERENCE.md §4`
3. **收敛自动**: 每 3 ADR / 5 SPEC → converge-scanner 扫描。→ `REFERENCE.md §5`
4. **会话恢复**: 新会话读 DESIGN-BOARD → 活跃 PHASE → 暂停 PHASE → 未读 REVIEW。→ `REFERENCE.md §6`
5. **看板一句话**: 用户说"看板" → 只读 DESIGN-BOARD + 活跃 PHASE + 未读 REVIEW，结构化呈现。→ `REFERENCE.md §20`
6. **并行可控**: 独立 SPEC 可同时施工，AI 检测文件重叠。→ `REFERENCE.md §23`
7. **ADR 链式**: 每个 ADR 声明依赖和被依赖。废弃时级联通知。→ `REFERENCE.md §24`
8. **实验不丢**: 改方向时 `git branch` 保存旧代码，不删除。→ `REFERENCE.md §22`

## 文件体系

在项目根创建 `docs/`（插件项目与 `.claude-plugin/` 同级）：

```
docs/
├── DESIGN-BOARD.md          # 持续更新
├── CONSTITUTION.md           # 几乎不变（含 NFR）
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

详细规则按需加载：`REFERENCE.md` — 包含阶段细节、冲突管理、收敛扫描、既有项目改造、质量门禁、拒绝处理、范围管理、Git 集成、模板使用、错误恢复、PHASE 状态机、并行施工、ADR 依赖链、NFR、里程碑、使用示例
