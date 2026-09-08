# dev-workflow

A disciplined, fully-traced software development pipeline for Claude Code / Claude Agent SDK — from requirements to shipped code, with every artifact on disk and every review handled by an independent sub-agent.

> 完整开发流水线技能：需求文档 → 子代理审核 → 修正 → 执行计划 → 子代理审核 → 修正 → 按计划编码 → 子代理 code review → 修改 → 测试，全程落文件到项目 `docs/dev/<任务>/`。

## 为什么需要这个技能

大多数 AI 编码会话的失败模式相同：没有落盘的产物、没有独立视角的审核、计划与代码漂移时无人察觉、教训在每次会话里被遗忘。`dev-workflow` 把一次开发任务变成一条**可追溯的流水线**：

- **每一步都要落文件** — 需求、计划、审核意见、处理结论全部写入 `docs/dev/<task-slug>/`，不允许只存在于对话里。
- **每次审核都是独立子代理** — 子代理使用全新上下文，prompt 自包含；只读不改，避免"作者-审稿人"的角色串扰。
- **门禁 (gate) 必须 P0 清零** — 需求未通过 → 不能进计划；计划未通过 → 不能编码；代码未通过 → 不能交付。
- **架构知识跨任务复用** — 前/后端架构缓存（`<项目>/.dev-workflow/*-architecture.md`）只盘点一次，10 天内有效。
- **技能本身会进化** — 每次运行结束会写教训到项目的 `lessons.md`，通用教训积累到阈值可固化回 SKILL.md。

## 安装

将本仓库的 `SKILL.md` 和 `references/` 目录放入你的 skills 目录即可。Claude Code 默认扫描以下位置（用户级 + 项目级）：

```
~/.claude/skills/dev-workflow/
└── SKILL.md
└── references/
    ├── review-prompts.md
    ├── frontend-context.md
    ├── backend-context.md
    └── lessons.md
```

或者项目级：

```
<your-project>/.claude/skills/dev-workflow/
```

重启 Claude Code / Claude Agent SDK 后，技能会自动加载。

## 触发条件

当用户提出**较大的开发任务**时自动触发 —— 新功能、跨多个文件、需要设计决策的改动 —— 或用户明确说"做个需求""实现 XX""开发 XX""走流程""先写需求文档/执行计划"，即使没明说"走流程"也应触发。

**不触发**：一句话能说清的小修、改错字、单点 bug 修复（走内置轻量路径）。

## 流水线阶段

```
              ┌─────────────────────┐
              │ 0. 判定档位与入口     │  全流程 vs 轻量；用户已带产物？
              └──────────┬──────────┘
                         ▼
        ┌──────────────────────────────┐
   ┌────┤ Phase A — 需求文档           │  docs/dev/<task>/requirements.md
   │    └──────────────────────────────┘
   │           │
   │           ▼
   │    ┌──────────────────────────────┐
   │    │ 审核需求（子代理，独立上下文） │  reviews/01-requirements.md
   │    └──────────────────────────────┘
   │           │
   │           ▼  P0 清零
   │    ┌──────────────────────────────┐
   │    │ Phase B — 执行计划           │  docs/dev/<task>/plan.md
   │    └──────────────────────────────┘
   │           │
   │           ▼  P0 清零
   │    ┌──────────────────────────────┐
   │    │ Phase C — 按计划编码          │  架构缓存命中判定
   │    └──────────────────────────────┘
   │           │
   │           ▼
   │    ┌──────────────────────────────┐
   │    │ Code Review（子代理）          │  reviews/03-code.md
   │    └──────────────────────────────┘
   │           │
   │           ▼
   │    ┌──────────────────────────────┐
   │    │ 测试（项目实际命令）          │
   │    └──────────────────────────────┘
   │           │
   │           ▼
   │    ┌──────────────────────────────┐
   │    │ 收尾汇报 + 进化反思            │  写 lessons.md
   │    └──────────────────────────────┘
```

每个阶段的"审核 → 修正"最多 **2 轮**。仍有 P0 未解决 → 停下向用户汇报卡点，不无限打磨。

## 文件结构

```
dev-workflow/
├── SKILL.md                      # 技能入口（精简骨架）
├── references/                   # 按需加载子文件
│   ├── review-prompts.md         # 三类审核的完整子代理 prompt 模板
│   ├── frontend-context.md       # 前端架构缓存的建档/刷新清单
│   ├── backend-context.md        # 后端架构缓存的建档/刷新清单
│   └── lessons.md                # 教训文档规范与模板
├── README.md                     # 本文件
├── LICENSE                       # MIT 许可证
└── .gitignore
```

**每次运行**产出的项目级文件：

```
<your-project>/
├── docs/dev/<task-slug>/
│   ├── requirements.md
│   ├── plan.md
│   ├── frontend-context.md       # 涉及前端时
│   ├── backend-context.md        # 涉及后端时
│   └── reviews/
│       ├── 01-requirements.md
│       ├── 02-plan.md
│       └── 03-code.md
└── .dev-workflow/                # 跨任务复用
    ├── frontend-architecture.md  # 前端架构缓存（≤10 天有效）
    ├── backend-architecture.md   # 后端架构缓存（≤10 天有效）
    └── lessons.md                # 本项目的所有教训（流程 + 项目专属）
```

## 核心护栏（不可削弱）

无论流水线怎么进化，下面四条规则永远保留：

1. **落文件铁律** — 需求、计划、审核意见、处理结论一律写入文件，不允许只存在于对话里。
2. **子代理独立审核** — 每个审核阶段都派全新上下文的子代理，prompt 必须自包含。
3. **P0 门禁** — 阻断级意见必须清零才能进入下一阶段。
4. **修正回合上限** — 每个阶段最多"审核 → 修正"2 轮，避免无限打磨。

## 轻量路径

当改动 ≤2 个文件、缺陷修复、报错排查时，走简化的轻量路径：

- 一份合并的需求 + 计划文档（`docs/dev/<task>/plan.md`，含目标和验收标准）
- 一次子代理审核
- 编码
- 测试

其余约定（落文件、独立审核、如实汇报测试结果）不变。

## 贡献

欢迎提交 Issue / PR。提交前请阅读 `SKILL.md` 第 10 节（进化）的护栏 —— SKILL.md 只允许**增量式修改**，四条核心护栏不得削弱或删除；超过 500 行时先固化去重再新增。

## 许可证

MIT — 见 [LICENSE](./LICENSE)。