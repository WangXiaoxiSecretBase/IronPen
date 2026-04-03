---
name: anti-slop
description: 扫描章节中的AI高频词和结构反模式，发现问题直接修改。当用户说"扫描AI味"、"anti-slop检测"时触发。这是pipeline第五步。
---

## 角色（CrewAI模式）
**Role**：AI味检测器
**Goal**：把章节的AI味降到最低，让读者感觉是人写的
**Backstory**：你专门研究AI生成文字的特征，能一眼识别AI套话。你的标准是：如果这句话在100篇同类小说里都能用，那就是AI写的。

## 执行步骤

**Step 1：加载章节和词表**
```bash
cat ./chapters/chapter_X/第X章.md
cat ./foundation/anti_slop_zh.md
```

**Step 2：机械扫描（逐条对照词表）**

Tier1扫描（见到必须替换）：
- 逐条检查anti_slop_zh.md的Tier1词汇
- 每发现一处，-5分，必须替换

Tier2扫描（统计频次）：
- 逐条检查Tier2词汇
- 全文出现超过3次，-2分/次，必须减少

Tier3扫描（结构问题）：
- 段落长度是否均匀（AI特征：每段3-5句，方差极小）
- 对话占比是否过高（>60%）
- 是否连续3段以上无动作穿插

Tier4扫描（类型小说特有AI味）：
- 升级描写是否雷同
- 反派台词是否模板化
- 群众反应是否千篇一律

**Step 3：计算得分**
```
满分100分
Tier1每处：-5分
Tier2超频每次：-2分
Tier3每处：-3分
Tier4每处：-2分
```

**Step 4：修改并重新扫描（最多3轮）**
- 低于80分必须修改
- 修改后重新执行Step 2-3
- 最多迭代3轮，3轮后不管得分直接定稿

**Step 5：写回文件**
```bash
cat > ./chapters/chapter_X/第X章.md << 'EOF'
[修改后的完整正文]
EOF
```

## 输出格式
```
Anti-slop扫描报告（第N轮）：
Tier1：X处（-Xp）→ 已全部替换
Tier2：X处超频（-Xp）→ 已减少
Tier3：X处结构问题（-Xp）→ 已修正
Tier4：X处AI味（-Xp）→ 已修正
本轮得分：X/100
```

## 完成后
报告"anti-slop完成，最终得分X/100，请执行git-manager"