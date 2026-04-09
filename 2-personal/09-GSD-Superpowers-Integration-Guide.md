# GSD + Superpowers 集成技术指南

> **版本:** 1.0
> **适用:** Claude Code 流程管理
> **依赖:** GSD v1.30+ / Superpowers v5.0+

---

## 1. 概述

### 1.1 集成哲学

GSD 和 Superpowers 各有所长，互补而非重叠：

| 维度 | GSD 负责范围 | Superpowers 负责范围 |
|------|-------------|---------------------|
| **管理粒度** | 项目级、阶段级 | 任务级、代码级 |
| **核心能力** | 范围控制、迭代管理、模块感知 | TDD 纪律、代码审查、验证门控 |
| **质量保证** | Goal-Backward 验证（读实际代码） | RED-GREEN-REFACTOR + 双重审查 |
| **持久化** | `.planning/` 全项目状态 | 设计文档、计划文档 |

**一句话总结：GSD 决定"做什么、不改什么"，Superpowers 决定"怎么做得好"。**

### 1.2 适用场景

- 需要持续迭代的项目（多个版本、多轮需求）
- 项目由多个模块组成，模块间有依赖关系
- 需求变更频繁，但必须确保变更不误伤无关模块
- 模块可能需要合并或拆分

### 1.3 核心取舍

本指南中一个关键决策：**不使用 `/gsd:execute-phase` 的自动执行模式**。

原因：GSD 的 `execute-phase` 会将计划交给 gsd-executor Agent 自主执行，虽然高效但绕过了 Superpowers 的 TDD 和审查纪律。改为手动逐任务执行，在每个任务上叠加 Superpowers 质量门控。

| 模式 | 速度 | 质量控制 | 适用场景 |
|------|------|---------|---------|
| 纯 GSD execute-phase | 快 | 中 | 原型开发、非关键功能 |
| GSD + Superpowers（本指南） | 中 | 高 | 生产代码、持续迭代 |
| 纯手动 | 慢 | 最高 | 核心模块、零容错 |

---

## 2. 前置条件

### 2.1 安装要求

```bash
# GSD 安装（npm）
npm install -g get-shit-done-cc

# Superpowers 安装（Claude Code 插件）
# 在 Claude Code 中运行 /install-plugin obra/superpowers
```

### 2.2 初始配置

```bash
# GSD 配置：推荐 balanced 配置
/gsd:set-profile balanced

# GSD 配置：开启质量 Agent
/gsd:settings
# → 开启 plan-checker
# → 开启 verifier
# → 开启 researcher（可选，复杂领域建议开启）
```

### 2.3 项目要求

- 必须是 Git 仓库
- 建议有清晰的模块目录结构
- 建议已有基础测试框架配置

---

## 3. 完整工作流

### 3.1 全局流程图

```
项目初始化（一次性）
  │
  ├── Step 1: /gsd:new-project
  └── Step 2: /gsd:map-codebase
        │
        ▼
  ┌── 迭代循环 ──────────────────────────────────────────┐
  │                                                        │
  │  Step 3: /gsd:new-milestone vX.X                      │
  │       │                                                │
  │       ▼                                                │
  │  ┌── 阶段循环 ──────────────────────────────────┐     │
  │  │                                               │     │
  │  │  GSD 规划层:                                  │     │
  │  │    Step 4:  /gsd:discuss-phase N              │     │
  │  │    Step 5:  /gsd:list-phase-assumptions N     │     │
  │  │    Step 6:  /gsd:plan-phase N                 │     │
  │  │       │                                       │     │
  │  │       ▼                                       │     │
  │  │  ┌── 任务循环 (Superpowers 质量层) ─────┐     │     │
  │  │  │                                       │     │     │
  │  │  │  Step 7:  /superpowers:tdd            │     │     │
  │  │  │           RED → GREEN → REFACTOR      │     │     │
  │  │  │                 │                     │     │     │
  │  │  │  Step 8:  /superpowers:verify         │     │     │
  │  │  │           新鲜证据 → 才能说完成        │     │     │
  │  │  │                 │                     │     │     │
  │  │  │  Step 9:  /superpowers:code-review    │     │     │
  │  │  │           修复 Critical/Important     │     │     │
  │  │  │                 │                     │     │     │
  │  │  │          下一个任务 ←──────────────────┘     │     │
  │  │  │                                              │     │
  │  │  GSD 验证层:                                    │     │
  │  │    Step 10: /gsd:verify-work N                  │     │
  │  │    Step 11: /gsd:validate-phase N               │     │
  │  │    Step 12: /gsd:progress                       │     │
  │  │       │                                         │     │
  │  │       ▼                                         │     │
  │  │  下一阶段 ←─────────────────────────────────────┘     │
  │  │                                                       │
  │  /gsd:progress → 下一迭代 ←─────────────────────────────┘
  │
  └── 持续循环...
```

---

### 3.2 阶段一：项目初始化（一次性）

#### Step 1: 建立项目结构

```
/gsd:new-project
```

**输入:** 对项目的高层描述
**输出:**

```
.planning/
├── PROJECT.md          # 项目定义、目标、约束
├── REQUIREMENTS.md     # 需求文档（含 v1/v2/out-of-scope 分层）
├── ROADMAP.md          # 阶段路线图
├── STATE.md            # 当前状态
└── config.json         # GSD 配置
```

**注意事项:**
- 在需求定义阶段，明确模块划分和模块间依赖关系
- 在路线图阶段，按模块组织阶段顺序
- 使用 `v1/v2/out-of-scope` 分层控制范围蔓延

#### Step 2: 生成模块地图

```
/gsd:map-codebase
```

**输入:** 无（自动分析代码库）
**输出:**

```
.planning/codebase/
├── stack.md            # 技术栈与依赖版本
├── architecture.md     # 架构模式与模块边界
├── structure.md        # 目录结构与文件职责
├── conventions.md      # 编码约定与设计模式
├── testing.md          # 测试策略与运行方式
├── integrations.md     # 模块间依赖与接口
└── concerns.md         # 已知问题与技术债
```

**注意事项:**
- 这是 Claude 对项目模块边界的认知基础，后续所有操作都依赖它
- 检查 `architecture.md` 和 `integrations.md`，确认模块边界描述准确
- 后续模块有重大变更时，重新运行此命令更新

---

### 3.3 阶段二：迭代启动

#### Step 3: 开启新一轮迭代

```
/gsd:new-milestone v2.0
```

**输入:** 里程碑名称和版本号
**输出:** ROADMAP.md 更新，新增本轮迭代的阶段列表

**提示词示例:**

```
新里程碑 v2.0：
- 用户模块：增加手机号登录
- 订单模块：重构结算流程
- 新增通知模块（从订单模块中拆出）
```

---

### 3.4 阶段三：GSD 规划层

#### Step 4: 讨论阶段内容

```
/gsd:discuss-phase 1
```

**输入:** 阶段编号
**输出:** `.planning/phases/01-xx/CONTEXT.md`

**关键操作:**
- 明确告诉 Claude 只涉及哪些模块
- 明确禁止修改的模块

**提示词示例:**

```
这个阶段要修改用户模块的登录逻辑：
- 涉及模块：user/, auth/
- 允许修改：user/service.py, auth/login.py
- 禁止修改：payment/, order/, notification/
- 目标：在现有用户名密码登录基础上增加手机号验证码登录
```

#### Step 5: 检查 Claude 的理解

```
/gsd:list-phase-assumptions 1
```

**输入:** 阶段编号
**输出:** 对话式输出（无文件生成）

**关键操作:**
- 重点检查 Claude 是否打算动不该动的文件
- 如果发现越界意图，回到 Step 4 纠正

**检查清单:**

- [ ] Claude 列出的文件是否都在允许范围内
- [ ] Claude 是否理解模块间的依赖关系
- [ ] Claude 的方案是否会影响其他模块的行为

#### Step 6: 生成执行计划

```
/gsd:plan-phase 1
```

**输入:** 阶段编号
**输出:** `.planning/phases/01-xx/XX-PLAN.md`

**关键操作:**
- 此时必须人工审查 PLAN.md
- 确认文件列表完全正确
- 确认任务粒度合理（每个任务 2-5 分钟工作量）

**PLAN.md 审查要点:**

```
# 检查项
1. 文件列表 → 是否都在允许范围内？
2. 任务顺序 → 依赖关系是否正确？
3. 测试要求 → 每个任务是否有对应的测试场景？
4. 验收标准 → 是否与阶段目标对齐？
5. 跨模块引用 → 是否有遗漏的导入/导出修改？
```

---

### 3.5 阶段四：Superpowers 质量层

对 PLAN.md 中的**每个任务**，按以下循环执行：

#### Step 7: TDD 纪律

```
/superpowers:test-driven-development
```

**作用:** 加载 TDD 纪律到上下文，此后所有代码操作遵循 RED-GREEN-REFACTOR

**铁律:**
> 先写了实现代码但没写测试 → 删除代码（不是注释，不是保留做参考，是删除）
> "应该可以" 不算通过 → 必须运行测试并确认输出

**然后指示 Claude 执行当前任务:**

```
执行 PLAN.md 的任务 N: [复制任务描述]
```

**执行过程:**

```
RED 阶段:
  1. 写一个最小失败测试，展示期望行为
  2. 运行测试 → 确认失败
  3. 确认失败原因正确（功能缺失，不是语法错误）

GREEN 阶段:
  1. 写最简代码让测试通过（YAGNI：不提前设计）
  2. 运行测试 → 确认通过
  3. 运行全量测试 → 确认没有破坏其他功能

REFACTOR 阶段:
  1. 消除重复代码
  2. 改善命名
  3. 提取通用函数
  4. 运行测试 → 确认仍然通过
```

**异常处理:**
- 如果遇到 bug → 进入 Step 10（调试流程）
- 如果测试框架问题 → 修复测试基础设施后继续

#### Step 8: 完成验证

```
/superpowers:verification-before-completion
```

**作用:** 在声称"任务完成"前，强制要求提供新鲜运行证据

**验证清单:**

```
1. 运行当前任务的测试 → 展示实际输出（不是"应该通过"）
2. 运行全量测试套件 → 确认无回归
3. 运行类型检查（如适用） → 确认无类型错误
4. 检查 git diff → 确认只修改了计划内的文件
```

**门控规则:**

| 检查项 | 结果 | 处理 |
|--------|------|------|
| 任务测试 | 通过 | 继续 |
| 任务测试 | 失败 | 回到 Step 7 GREEN 阶段 |
| 全量测试 | 通过 | 继续 |
| 全量测试 | 失败 | 回到 Step 7，修复回归 |
| 文件范围 | 越界 | **立即停止**，检查 plan 并调整 |
| 文件范围 | 符合 | 继续 |

#### Step 9: 代码审查

```
/superpowers:requesting-code-review
```

**作用:** 派出 code-reviewer 子 Agent 审查当前任务的代码

**审查维度:**

- 规格合规性：代码是否与 PLAN.md 中的任务描述一致
- 代码质量：可读性、复杂度、命名
- 架构一致性：是否遵循项目现有模式
- 安全性：是否引入安全隐患
- 测试覆盖：测试是否充分

**反馈处理规则:**

| 严重级别 | 处理方式 |
|----------|---------|
| Critical | 立即修复，修复后重新审查 |
| Important | 当前任务内修复 |
| Suggestions | 记录，可延后处理 |

**修复后:**
- 修复代码
- 回到 Step 8 重新验证
- 如有 Critical 修复，重新 Step 9 审查

**任务完成标志:**

```
✅ Step 7 TDD 循环完成（RED → GREEN → REFACTOR）
✅ Step 8 验证通过（新鲜证据确认）
✅ Step 9 审查通过（无 Critical/Important 未修复）
→ 进入下一个任务，回到 Step 7
```

---

### 3.6 阶段五：GSD 验证层

所有 PLAN.md 中的任务完成后：

#### Step 10: 功能交付验证

```
/gsd:verify-work 1
```

**输入:** 阶段编号
**输出:** 对话式 UAT 验证

**核心机制:**
- 从 SUMMARY.md 中提取可测试的交付物
- 逐个呈现测试场景
- 自动诊断失败并创建修复计划
- **Goal-Backward: 读实际代码验证，不信任 SUMMARY 声明**

**重点检查:**
- 是否只修改了目标模块的文件
- 是否有意外的副作用
- 跨模块接口是否仍然正常

#### Step 11: 测试覆盖验证

```
/gsd:validate-phase 1
```

**输入:** 阶段编号
**输出:** VALIDATION.md 更新

**作用:**
- 检查 Nyquist 验证缺口
- 自动补充缺失的测试
- 确保测试覆盖了所有验收标准

#### Step 12: 进度确认

```
/gsd:progress
```

**输出:**
- 可视化进度条
- 完成百分比
- 当前状态摘要
- 关键决策记录
- 未解决问题列表

**判断:**

```
进度 100% 且验证通过 → 进入下一阶段（回到 Step 4）
进度未完成 → 检查是否有遗漏任务
```

---

## 4. 特殊场景

### 4.1 模块拆分

当需要将一个模块拆分为多个子模块时：

```
Step A: /gsd:discuss-phase N
提示词:
  "将用户模块拆分为认证模块(auth/)和授权模块(rbac/)。
   列出所有需要移动的文件。
   列出所有跨模块引用需要更新的地方。
   禁止修改 payment/ 和 order/ 目录下的任何文件。"

Step B: /gsd:list-phase-assumptions N
→ 重点检查：是否有遗漏的导入路径更新

Step C: /gsd:plan-phase N
→ 人工审查：确认文件移动清单完整

Step D: 逐任务执行（Step 7-9 Superpowers TDD）
→ 每移动一组文件后立即运行全量测试

Step E: /gsd:validate-phase N
→ 全量测试覆盖确认

Step F: /gsd:map-codebase
→ 重新生成模块地图，反映新的模块结构
```

### 4.2 模块合并

当需要将多个模块合并为一个时：

```
Step A: /gsd:discuss-phase N
提示词:
  "将 payment-alipay/ 和 payment-wechat/ 合并为 payment/ 模块。
   统一接口设计。
   保留两个渠道的独立配置能力。
   禁止修改 user/ 和 order/ 目录下的任何文件。"

Step B-C: 同模块拆分

Step D: 逐任务执行
→ 优先合并接口层 → 然后合并实现层 → 最后更新导入

Step E-F: 同模块拆分
```

### 4.3 紧急 Bug 修复

当迭代进行中需要紧急修复线上 bug 时：

```
方案一（最轻量）: 不影响当前迭代
  /gsd:fast "修复用户模块登录超时问题"
  → 直接修复，提交，不经过 Superpowers

方案二（需质量保证）:
  /gsd:insert-phase 3.1 "紧急修复: 用户模块登录超时"
  → 在当前阶段之间插入 decimal phase
  → 按正常流程 Step 4-12 执行

选择标准:
  线上 P0 故障 → 方案一（速度优先）
  P1-P2 问题   → 方案二（质量优先）
```

### 4.4 跨模块需求

当需求涉及多个模块的协调修改时：

```
Step 1: /gsd:discuss-phase N
→ 明确列出涉及的所有模块和修改范围

Step 2: /gsd:plan-phase N
→ 计划中明确标注每个任务属于哪个模块
→ 标注模块间的修改顺序（被依赖方先改）

Step 3: 执行时按模块分组
→ 先完成被依赖模块的修改
→ 每个模块内走 Step 7-9 Superpowers 质量门控
→ 模块切换时运行全量测试

Step 4: /gsd:verify-work N
→ 重点验证模块间接口是否正确对接
```

---

## 5. 会话管理

### 5.1 中断恢复

开发过程中 Claude 会话可能因上下文窗口限制而中断：

```
# 中断前（主动）
/gsd:pause-work
→ 创建 .continue-here 文件
→ 更新 STATE.md

# 恢复后（新会话）
/gsd:resume-work
→ 读取 STATE.md
→ 检测 .continue-here 文件
→ 提供上下文感知的下一步建议

# 恢复后需要做的
1. 重新加载 Superpowers TDD 纪律: /superpowers:test-driven-development
2. 检查当前任务状态: 读取 PLAN.md 中已完成的任务
3. 从中断的任务继续执行
```

### 5.2 模块地图更新

以下情况需要重新运行 `map-codebase`：

- 完成模块合并或拆分后
- 新增模块后
- 大规模重构后
- 里程碑完成时

```
/gsd:map-codebase
→ 更新 .planning/codebase/ 下的 7 个文件
→ Claude 对项目边界的认知同步更新
```

---

## 6. 三层防护体系总结

本集成方案形成三层防护，确保变更精确且高质量：

```
┌─────────────────────────────────────────────────────────┐
│  第一层: GSD 规划防护                                     │
│  ─────────────────────                                   │
│  • map-codebase 提供模块边界认知                          │
│  • discuss-phase 明确修改范围和禁止范围                    │
│  • plan-phase 精确到文件级别的执行计划                     │
│  • list-phase-assumptions 提前发现越界意图                │
│                                                          │
│  防护目标: 确保计划正确（改对的文件，不改错的文件）         │
├─────────────────────────────────────────────────────────┤
│  第二层: Superpowers 质量防护                             │
│  ─────────────────────────                               │
│  • TDD: RED-GREEN-REFACTOR 确保代码有测试覆盖            │
│  • verification: 新鲜证据门控，禁止模糊声称               │
│  • code-review: 双重审查（规格合规 + 代码质量）           │
│  • git diff 检查: 确认实际修改在计划范围内                 │
│                                                          │
│  防护目标: 确保执行正确（代码质量高，无意外修改）          │
├─────────────────────────────────────────────────────────┤
│  第三层: GSD 验证防护                                     │
│  ─────────────────────                                   │
│  • verify-work: Goal-Backward 验证，读实际代码            │
│  • validate-phase: 测试覆盖缺口检测与补充                 │
│  • progress: 整体状态可视化与未完成项追踪                  │
│                                                          │
│  防护目标: 确保交付正确（目标达成，无回归，无遗漏）        │
└─────────────────────────────────────────────────────────┘
```

---

## 7. 快捷操作参考

### 7.1 日常场景速查

| 场景 | 命令 | 质量级别 |
|------|------|---------|
| 线上紧急修复 | `/gsd:fast "描述"` | 低（无 Superpowers） |
| 单模块小功能 | `/gsd:quick "描述" --discuss` | 中（GSD 受控） |
| 单模块大功能 | Step 4 → Step 6 → Step 7-9 → Step 10-12 | 高（全流程） |
| 跨模块修改 | Step 4 → Step 6 → Step 7-9（按模块分组）→ Step 10-12 | 高（全流程） |
| 模块合并/拆分 | Step 4 → Step 6 → Step 7-9 → Step 10-12 + 重新 map-codebase | 高（全流程 + 地图更新） |

### 7.2 Skill 调用速查表

| 阶段 | Skill | 何时调用 |
|------|-------|---------|
| 初始化 | `/gsd:new-project` | 项目首次创建 |
| 初始化 | `/gsd:map-codebase` | 首次或模块结构变化后 |
| 迭代 | `/gsd:new-milestone vX.X` | 每轮新迭代 |
| 规划 | `/gsd:discuss-phase N` | 每个阶段开始 |
| 规划 | `/gsd:list-phase-assumptions N` | 讨论后、计划前 |
| 规划 | `/gsd:plan-phase N` | 确认理解后 |
| 执行 | `/superpowers:test-driven-development` | 每个任务开始 |
| 执行 | `/superpowers:verification-before-completion` | 每个任务完成前 |
| 执行 | `/superpowers:requesting-code-review` | 每个任务验证后 |
| 执行 | `/superpowers:systematic-debugging` | 遇到 bug 时 |
| 验证 | `/gsd:verify-work N` | 所有任务完成后 |
| 验证 | `/gsd:validate-phase N` | verify-work 后 |
| 管理 | `/gsd:progress` | 任何时候查看进度 |
| 管理 | `/gsd:pause-work` | 中断前 |
| 管理 | `/gsd:resume-work` | 恢复时 |
| 管理 | `/gsd:insert-phase N.M` | 插入紧急阶段 |
| 管理 | `/gsd:ship N` | 创建 PR |

---

## 8. 注意事项

### 8.1 已知限制

1. **不支持 GSD 自动执行**: 必须手动逐任务执行才能叠加 Superpowers
2. **会话恢复需重新加载**: 恢复会话后需重新调用 `/superpowers:test-driven-development`
3. **上下文窗口消耗**: 两个框架的 Skill 同时加载会占用较多上下文，长任务注意 `pause-work`
4. **plan-phase 冲突**: Superpowers 也有 `writing-plans`，本方案中统一使用 GSD 的 `plan-phase`

### 8.2 最佳实践

1. **模块地图是关键**: `map-codebase` 的质量直接决定 Claude 的模块感知能力，务必确保准确
2. **人工审查计划**: `plan-phase` 生成的 PLAN.md 是整个流程的锚点，务必逐文件检查
3. **git diff 是最终防线**: 每个任务完成后检查 `git diff --name-only`，确认修改范围
4. **小步提交**: 每个任务完成后单独提交，便于回滚
5. **定期更新地图**: 模块结构变化后及时重新 `map-codebase`

### 8.3 故障排除

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| Claude 修改了不相关的文件 | 模块地图过时 | `/gsd:map-codebase` 更新 |
| 测试总是失败 | 测试框架配置问题 | 先修复测试基础设施 |
| Claude 不遵循 TDD | TDD Skill 未加载 | 重新调用 `/superpowers:test-driven-development` |
| 计划中的文件范围过大 | discuss-phase 不够具体 | 回到 Step 4，更明确地限定范围 |
| 会话中断后行为异常 | 状态未持久化 | `/gsd:pause-work` + `/gsd:resume-work` |
