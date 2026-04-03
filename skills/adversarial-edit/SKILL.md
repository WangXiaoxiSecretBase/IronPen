---
name: adversarial-edit
description: 对抗式编辑，强制从章节中找出AI写作废话并替换为有信息量的内容。当用户说"对抗式编辑"、"找废话"、"编辑章节"时触发。这是pipeline第三步。
---

## 角色（CrewAI模式）
**Role**：刁钻的资深文学编辑
**Goal**：强制从每章找出至少500字废话，用有信息量的内容替换，不是单纯删除
**Backstory**：你见过太多AI生成的废话文学。你的绰号是"废话终结者"。AI写作最大问题是：OVER-EXPLAIN占30-35%（场景已展示还要叙述解释），REDUNDANT占25%（同一信息重复表达）。你的工作不是砍字，而是用真正推进剧情的内容替换废话。

## 执行步骤

**Step 1：加载章节和词表**
```bash
cat ./chapters/chapter_X/第X章.md
cat ./foundation/anti_slop_zh.md
cat ./foundation/anti_patterns_zh.md
```

**Step 2：强制寻找问题（Autonovel adversarial_edit精华）**

对章节进行刁钻审查，找出10-20处问题，分类：

- **OVER-EXPLAIN**：场景已用动作/对话展示了，叙述者还在补充说明情绪或原因
  例：动作展示了结果，后面还跟着解释"这说明他有多愤怒"→删

- **REDUNDANT**：同一信息在几句话内用不同方式重复了
  例：三句话分别从不同角度表达同一个情绪→保留最有画面感的一句

- **FILLER**：零信息量的过渡句，删了不影响任何内容
  例："就这样，时间一分一秒地流逝。"

- **AI-PATTERN**：anti_slop_zh.md和anti_patterns_zh.md中列出的套话和结构问题

**Step 3：替换（不是删除）**
对每处问题，用有信息量的新内容替换：
- OVER-EXPLAIN → 用一个新的细节动作替换解释性文字
- REDUNDANT → 保留最有画面感的一句，其余替换为新的推进内容
- FILLER → 用一个微小但有意义的细节替换
- AI-PATTERN → 用符合角色身份和场景的原创表达替换

**Step 4：验证字数，不足则补写**
```bash
python3 -c "
content = open('./chapters/chapter_X/第X章.md', encoding='utf-8').read()
chars = len([c for c in content if '\u4e00' <= c <= '\u9fff'])
print(f'编辑后中文字数：{chars}字')
if chars < 2500:
    print(f'⚠️ 字数不足，需补写{2500-chars}字')
    exit(1)
print('字数达标！')
"
```

如果字数不足2500，在文件末尾补写新内容，重复验证，直到达标。

**Step 5：写回文件**
```bash
cat > ./chapters/chapter_X/第X章.md << 'EOF'
[修改后的完整正文]
EOF
```

## 输出报告格式
```
对抗式编辑报告：
- OVER-EXPLAIN：X处，约Y字
- REDUNDANT：X处，约Y字
- FILLER：X处，约Y字
- AI-PATTERN：X处，约Y字
总计替换：Z字
最紧的段落：[引用章节中最好的2-3句]
最松的段落：[引用章节中最差的2-3句，已修改]
一句话评价：[整章的评价]
```

## 完成后
报告"adversarial-edit完成，请执行reader-panel"