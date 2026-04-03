# IronPen 🖊️

> 基于 OpenClaw + 阶跃星辰 API 的全自动中文网文生成系统

---

## 项目简介

IronPen 是一个全自动中文网文生成流水线，每天凌晨 3 点自动生成一章约 2500 字的男频都市爽文，并通过飞书通知作者检查后上传番茄小说。

**当前连载作品**：《出狱觉醒医武双修》
- 主角：林战，28岁，蒙冤入狱三年，出狱觉醒医武双修系统
- 类型：男频都市爽文
- 平台：番茄小说

---

## 架构说明

IronPen 融合了四个开源框架的精华：

| 框架 | 借鉴内容 |
|------|---------|
| [Autonovel](https://github.com/NousResearch/autonovel) | 对抗式编辑、读者面板、evaluate→commit/rollback 循环 |
| AuthorClaw | SOUL.md + STYLE-GUIDE.md + VOICE-PROFILE.md 三层人格过滤 |
| Novelcrafter | characters.md + canon.md Codex 知识图谱 |
| CrewAI | Role-Goal-Backstory 角色定义模式 |

**运行环境**：
- Agent 框架：飞书妙搭 OpenClaw（云端）
- 模型：阶跃星辰 step-3.5-flash
- 通知渠道：飞书

---

## Pipeline 流程

每天凌晨 3 点，OpenClaw 按顺序执行 7 个 skill：

```
novel-planner
  → 读取 outline.md + canon.md + state.md
  → 生成本章节拍（开头钩子/爽点布局/结尾钩子）
  → 写入 beats_第X章.md

novel-generator
  → 加载三层人格过滤（SOUL + STYLE-GUIDE + VOICE-PROFILE）
  → 加载 Codex（characters + canon）
  → 生成 2500+ 字正文
  → exec 验证字数，不足自动补写

adversarial-edit
  → 强制找出 OVER-EXPLAIN / REDUNDANT / FILLER / AI-PATTERN
  → 用有信息量的内容替换废话（不是单纯删除）

reader-panel
  → 王大锤 / 李编辑 / 陈大神 / 小美 4 人格评审
  → 找 3/4 共识问题并直接修改

anti-slop
  → Tier1-4 词表扫描 + 结构反模式检测
  → 得分 < 80 必须修改

git-manager
  → 综合评分（爽点/钩子/anti-slop/文风各 25 分）
  → 分数提升 → IronPen git commit
  → 分数下降 → git rollback 重来
  → 连续 2 轮不提升 → plateau 定稿

novel-summarizer
  → 提取摘要更新 state.md
  → 新设定追加 canon.md
  → 角色更新 characters.md
  → 矛盾检测
  → rsync 同步到 IronPen → git push
  → 飞书通知
```

---

## 文件结构

```
~/workspace/agent/                  ← OpenClaw 工作目录
├── workspace/
│   ├── SOUL.md                     ← 铁笔先生人格
│   ├── AGENTS.md                   ← 启动序列
│   ├── HEARTBEAT.md                ← 每日检查清单
│   ├── foundation/                 ← 设定文件
│   │   ├── STYLE-GUIDE.md          ← 写作规则（AuthorClaw 第二层）
│   │   ├── VOICE-PROFILE.md        ← 文风指纹（AuthorClaw 第三层）
│   │   ├── canon.md                ← 硬设定数据库（Novelcrafter Codex）
│   │   ├── characters.md           ← 角色 Codex 追踪表
│   │   ├── outline.md              ← 故事大纲（三卷）
│   │   ├── state.md                ← 当前进度
│   │   ├── world.md                ← 世界观+系统等级体系
│   │   ├── anti_slop_zh.md         ← 中文禁用词表（Tier1-5）
│   │   └── anti_patterns_zh.md     ← 结构反模式库（28种）
│   └── chapters/                   ← 生成的章节
│       ├── beats_第X章.md          ← 节拍文件
│       └── 第X章.md                ← 正文文件
└── skills/                         ← 7 个写作 skill
    ├── novel-planner/SKILL.md
    ├── novel-generator/SKILL.md
    ├── adversarial-edit/SKILL.md
    ├── reader-panel/SKILL.md
    ├── anti-slop/SKILL.md
    ├── git-manager/SKILL.md
    └── novel-summarizer/SKILL.md

~/IronPen/                          ← GitHub 备份仓库
├── foundation/
├── chapters/
├── skills/
└── README.md
```

---

## 使用方法

### 手动触发单步
在飞书对话里发：
```
使用 novel-planner 规划第 X 章
使用 novel-generator 生成第 X 章
使用 adversarial-edit 处理第 X 章
```

### 手动触发完整 pipeline
```
请按顺序执行今日写作 pipeline：
novel-planner → novel-generator → adversarial-edit → reader-panel → anti-slop → git-manager → novel-summarizer
```

### 自动运行
OpenClaw cron 每天凌晨 3 点（Asia/Shanghai）自动触发完整 pipeline，完成后发飞书通知。

---

## 灵感来源

- [NousResearch/autonovel](https://github.com/NousResearch/autonovel)
- [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- AuthorClaw（OpenClaw 写作专用 fork）
- Novelcrafter（商业小说写作平台）
- CrewAI（多 agent 框架）
