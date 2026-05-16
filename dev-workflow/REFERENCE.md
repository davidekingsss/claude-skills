# Dev Workflow Reference

> 详细规则参考。核心行为规则见 `SKILL.md`。

## 1. 阶段感知细节

### 1.1 多信号优先级

冲突 > 收敛 > 澄清 > 施工 > 头脑风暴。用户说"先做着"→ 施工优先。

### 1.2 追问上限

每次头脑风暴最多 5 个关键问题。追问完后给出口："这些够我开始了，要继续深入还是先记录？"

### 1.3 多想法处理

一次对话多个独立想法 → 各建一个 BRIEF，DESIGN-BOARD 标注关联。
一个想法衍生多个子方向 → 一个 BRIEF + 子节，澄清阶段再决定是否拆分。

## 2. 命名与编号规范

`{TYPE}-{NNN}-{slug}.md`。编号自动递增（扫描目录取最大 +1）。日期格式 `YYYY-MM-DD`。

与已有 DDR 体系并存：沿用现有编号，不重排。

## 3. 流转规则

1. 一句话可能触发多阶段 → 按优先级
2. 不跳过阶段（除非用户明确要求）
3. 不阻塞用户（"先做着"→ 施工再补文档）
4. 渐进落笔：BRIEF → ADR → SPEC
5. BRIEF > 30 天未推进 → 会话开始时间用户"要不要继续"

## 4. 冲突管理

### 4.1 检测

1. 静默扫描 DESIGN-BOARD + 所有 ADR
2. 筛选相关 ADR：关键词匹配 + 模块匹配 + 明确引用
3. 调 conflict-resolver agent

### 4.2 分类

- 轻微冲突（参数调整）→ 直接更新 ADR，告知用户
- 根本冲突（方向矛盾）→ agent 分析影响范围，呈现给用户

### 4.3 施工中遇冲突

1. PAUSE PHASE → 状态"暂停: 等待冲突解决"
2. 冲突解决后：推翻 → 废弃 PHASE + 新建；并存 → 恢复 PHASE；撤回 → 恢复 PHASE

### 4.4 用户选项

- 推翻旧决策 → ADR 标记 superseded，受影响 SPEC 重做
- 并存 → 缩小旧 ADR 范围 + 新建 ADR
- 撤回 → 保留旧 ADR，新想法暂存 BRIEF

## 5. 架构收敛

| 触发 | 动作 |
|------|------|
| 每 3 ADR | 扫描关联，提出合并/抽象建议 |
| 每 5 SPEC | 启动 converge-scanner agent |
| 冲突解决后 | 更新 DESIGN-BOARD 关联 |

### 5.1 代码摘要

为 agent 准备：ARCHITECTURE.md 模块入口 50 行 + SPEC 关联模块的函数签名。每模块 < 200 tokens。

### 5.2 通知

REVIEW 写入后即时告知 + 下次会话提醒未读 REVIEW。

## 6. 跨会话上下文恢复

新会话启动序列：

```
1. 读 DESIGN-BOARD.md → 项目全局状态
2. 读所有 PHASE-*.md → 活跃/暂停 Phase
3. 活跃 Phase → 读关联 SPEC → 继续
4. 暂停 Phase → 告知用户"上次暂停了，继续？"
5. 未读 REVIEW → 提醒
6. > 30 天未推进 BRIEF → "要继续讨论吗？"
```

## 7. 既有项目改造

### 7.1 反向工程

```
Step 1: 扫描项目根 → 识别类型
Step 2: 读 CLAUDE.md/README.md → 提取原则
Step 3: 读主入口 → 提取模块
Step 4: 读 config → 提取技术栈
Step 5: 生成 DESIGN-BOARD / CONSTITUTION / ARCHITECTURE
Step 6: 向用户确认
Step 7: 用户修正 → 定稿
```

### 7.2 单文档迁移

1. 解构：读全文 → 提取为 CONSTITUTION + ARCHITECTURE + ADR-*.md
2. 保留原文档，新建 `docs/` 并行结构
3. 渐进迁移：每次施工时迁移涉及的部分
4. 原文档顶部提示"已迁移至 docs/，本文档保留为历史参考"

## 8. 文件体系

```
docs/
├── DESIGN-BOARD.md
├── CONSTITUTION.md
├── ARCHITECTURE.md
├── briefs/        BRIEF-NNN-slug.md
├── adr/           ADR-NNN-slug.md
├── specs/         SPEC-NNN-slug.md
├── phases/        PHASE-NNN-slug.md
├── milestones/    MILESTONE-NNN-vX.Y.Z.md
└── reviews/       REVIEW-YYYY-MM-DD.md
```

## 9. 激活与路由机制

通过 description 中的语义类别触发加载（代码修改、功能开发、bug修复、架构设计等，不再依赖固定关键词）。加载后执行路由决策：根据变更规模（微小/小型/中型/大型）选择对应路径。纯信息询问不进入任何路径。详细规模判断见 §28。

## 10. 质量门禁

| 转换 | 条件 |
|------|------|
| 头脑风暴 → 澄清 | BRIEF 有 ≥1 核心问题 + ≥1 初设方向 |
| 澄清 → 固化 | ≥1 明确决策点，用户看过选项 |
| 固化 → 施工 | ADR 已采纳，SPEC 清单可逐项验证 |
| 施工 → 验证 | PHASE 已完成，验收标准全勾选 |

门禁不阻塞——用户说"先做着"可跳过，DESIGN-BOARD 标记即可。

## 11. 用户拒绝处理

- 不反复提议同一方案 → DESIGN-BOARD 标记"已讨论，用户拒绝"
- 提供替代方案
- 三个方案都被拒 → "先放一放，了解更多后再定？"
- 被拒文档不删除 → 标记"已拒绝: 日期"

## 12. 范围管理

- BRIEF ≥3 独立方向 → 拆分多个 BRIEF
- SPEC 清单 > 10 项 → 拆多 PHASE（每 3-5 项）
- 范围蔓延 → AI 提醒"从 X 扩展到了 YZ，要不要先做 X？"

## 13. Git 集成

| 时机 | Commit |
|------|--------|
| 新建/修改 docs/ | `docs: ...` |
| ADR 状态变更 | `adr: ADR-003 已采纳` |
| PHASE 完成 | `feat: xxx (PHASE-001)` |
| REVIEW 生成 | `review: 收敛扫描 YYYY-MM-DD` |

文档和代码分开 commit。

## 14. 使用示例

### 新项目

```
用户: "我想做个命令行工具，批量下载论文 PDF 提取文字"

AI: [头脑风暴] → BRIEF-001-pdf-downloader.md
    "确认几个关键点: 1.来源? 2.提取后干什么? 3.自己用还是给别人?"
    [≤5 问]

用户: "PubMed，自己用，全文搜索"

AI: [澄清] "技术方案: A) Python+SQLite FTS5 B) Elasticsearch。倾向哪个?"

用户: "A"

AI: [固化] → ADR-001-sqlite-fts5.md "已记录。要施工还是继续讨论？"
```

### 既有项目

```
用户: "factory 要加用户认证"

AI: [读 DESIGN-BOARD + ADR] "当前 14 ADR 已施工。
     认证是全新方向 → BRIEF-015-auth.md。
     可能影响 worker 调用 → 标记为收敛待检查。"
```

## 15. (已移除 — 变更规模路由已替代旧的不适用场景分类，见 §28)

## 16. 模板使用说明

### 16.1 查找

从 SKILL.md 路径推导：`{SKILL.md 所在目录}/templates/{TEMPLATE}.md`

### 16.2 使用

1. 复制模板 → 替换 `{{placeholder}}`
2. 不保留占位符，不适用写 `N/A`
3. 文件名 `{TYPE}-{NNN}-{slug}.md`
4. 日期 `YYYY-MM-DD`
5. 可增加字段，不删除核心字段

## 17. 错误恢复

1. 创建父目录
2. 原子写入（临时文件 → rename）
3. 关联更新滞后 → 下次会话自动补注册孤立文件
4. 不静默失败 → 告知用户

## 18. docs/ 目录位置

- 新项目：Git 仓库根下
- 插件项目：与 `.claude-plugin/` 同级
- 项目根识别：`.git/`、`package.json`、`CLAUDE.md` 所在目录

## 19. PHASE 状态机

```
待开始 → 施工中 → 已完成
            │
            ├→ 暂停（用户中断/冲突等待）
            │     ├→ 施工中（恢复）
            │     └→ 废弃（推翻）
            └→ 废弃（放弃/superseded）
```

## 20. 看板视图

用户说"看板"→ 读 DESIGN-BOARD + 活跃 PHASE + 未读 REVIEW，输出：

```
=== 项目看板 ===
📋 已识别: P1 BRIEF-015, P2 BRIEF-016
✅ 已决策: 14 ADR
🏗️ 施工中: 无
🔔 REVIEW-2026-05-15 (未读)
下一步: 讨论 BRIEF-015 技术方案
```

## 21. 测试集成

SPEC 验收标准至少一项测试条目。PHASE 完成标准包含"测试通过"。测试是施工的一部分，非独立阶段。

## 22. 施工中改方向

1. `git branch experiment/SPEC-XXX-old` 保存旧代码
2. 主分支回退
3. PHASE 标记"废弃: ADR 推翻"，记录旧分支名
4. 不删除旧代码
5. 基于新 ADR 创建新 SPEC

## 23. 并行施工

### 条件

- 无文件重叠（AI 通过 SPEC 关联模块判断）
- 无数据依赖
- 用户知情

### 检测

启动新 PHASE 前检查所有活跃 PHASE 的影响文件。重叠 → 建议串行。

## 24. ADR 依赖链

ADR 模板包含：`依赖` + `被依赖`（AI 自动计算）。

级联规则：ADR 被 superseded → 扫描依赖字段 → 受影响 ADR 标记"待审查" → 用户逐个处理。

循环依赖检测：新建 ADR 时检查，不允许循环。

## 25. 非功能需求（NFR）

CONSTITUTION 中记录性能/安全/可用性要求。SPEC 验收标准中转化为可测量条件。

## 26. 里程碑

### 文件格式

`docs/milestones/MILESTONE-NNN-vX.Y.Z.md`:

```markdown
# MILESTONE-NNN: vX.Y.Z — 标题
> 目标日期 | 状态: 规划中 | 施工中 | 已发布

## 包含 SPEC
| SPEC | 功能 | 状态 |

## 发布检查
- [ ] 所有 SPEC 验收通过
- [ ] 测试全部通过
- [ ] 文档更新
- [ ] Git tag
```

### 看板集成

DESIGN-BOARD 底部显示：`🏷️ v0.1.0 ████████░░ 2/3 SPEC`

## 28. 变更规模详细判断标准

### 28.1 微小 (Tiny)

触发条件（满足全部）：
- 影响 ≤1 文件
- 无新依赖、无 API/接口变更
- 用户意图无歧义（不需要追问设计选择）
- 示例：改按钮颜色、修 null 检查、加一行日志、修 typo

### 28.2 小型 (Small)

触发条件（满足全部）：
- 影响 ≤3 文件
- 无新外部依赖
- 选择明确（如"用 A 库还是 B 库"不在此列）
- 示例：加一个校验中间件、优化 SQL 查询、加单个 API 端点

### 28.3 中型 (Medium)

触发条件（满足任一）：
- 影响 ≥4 文件或多模块
- 涉及新技术/模式选择
- 存在明确的方案取舍
- 示例：加认证系统、重构模块、换数据库访问层

### 28.4 大型 (Large)

触发条件（满足任一）：
- 跨系统/跨服务
- 架构层面变更
- 破坏性 API 变更
- 示例：框架迁移、拆微服务、新子系统

### 28.5 边界规则

- 不确定规模 → 取大不取小（宁可多记录不可漏记录）
- 用户说"太小了" → 降低一级
- 用户说"先做着" → 降为微小并执行，事后补文档
- 微小变更在同文件叠加 ≥3 次 → AI 主动建议提升为小型

## 29. CHANGES.md 格式

`docs/CHANGES.md` 是微小变更的汇总日志。由 AI 自动维护。

```markdown
# 变更日志
> 微小变更汇总。中大型变更见 DESIGN-BOARD.md 和各 SPEC。

## YYYY-MM
- [DD] 变更描述 (影响文件)
```

规则：
- 按月分组，同月追加到已有 `## YYYY-MM` 节
- 格式：`- [日期] 描述 (文件)`
- AI 在微小变更后自动追加，不询问
- 项目无 `docs/` 时跳过记录

## 30. 小型变更 SPEC 简化规则

小型变更使用标准 SPEC 模板，但：
1. "关联 ADR" 填 `N/A`（跳过 ADR 流程）
2. "关联模块" 填 `N/A`（不分析模块影响）
3. "目标" + "施工清单" + "验收标准" 为必填
4. "输入/输出" 按需填写
5. 命名：`SPEC-NNN-slug.md`（与标准 SPEC 相同命名规则）
6. DESIGN-BOARD 在"已施工"表标注为小型变更（并列显示，便于区分）

## 31. PR 自动化配置

### 31.1 钩子文件

在项目根创建 `.claude/workflow-hooks/` 目录：

```
.claude/workflow-hooks/
├── post-merge.sh     # PR 合并后执行
├── post-release.sh   # 里程碑发布后执行（可选）
└── README.md         # 钩子说明（可选）
```

### 31.2 post-merge.sh 约定

```bash
#!/bin/bash
# 在 PR 合并后由 AI 调用
# 工作目录：项目根
# 环境变量：SPEC_ID, PHASE_ID, PR_NUMBER, BRANCH
# 沙箱约束：仅可写项目树内路径，仅可访问 localhost

set -e
# 示例：打包产物
# tar -czf release-${PHASE_ID}.tar.gz ./dist/
# 示例：同步到外部（需用户预先授权路径）
# rsync -av ./dist/ /authorized/external/path/
```

### 31.3 触发流程

1. AI 检测 PHASE 完成 → 提示用户"是否创建 PR？"
2. 用户确认 → `gh pr create --title "feat: ${TITLE} (${PHASE_ID})" --body "$(SPEC 摘要)"`
3. AI 在 PHASE 中记录 PR URL
4. 后续会话检测 PR 已合并 → 执行 `post-merge.sh`（若存在）
5. AI 更新 PHASE 状态为"已完成 + 已合并"
6. 若所有关联 SPEC 完成 → 提示用户"是否发布里程碑？"

### 31.4 沙箱合规

- `gh pr create` / `gh pr merge`：默认允许（git 操作）
- `git tag` / `git push`：默认允许
- `post-merge.sh` 内容由用户预先编写，AI 仅执行不修改
- 外部同步：若脚本中需要访问外部网络/路径，须用户在 Cowork 中授予权限
- 若沙箱拒绝脚本执行 → AI 告知用户具体被拒绝的操作，不静默失败

### 31.5 无钩子时的默认行为

若 `.claude/workflow-hooks/post-merge.sh` 不存在：
- PR 合并后仅更新 PHASE 状态
- 里程碑完成时执行 `git tag vX.Y.Z`
- 不做任何外部同步
