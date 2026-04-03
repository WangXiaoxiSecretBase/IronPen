---
name: git-manager
description: 章节综合评分和git回滚循环。当用户说"评分"、"提交章节"、"git管理"时触发。这是pipeline第六步，实现Autonovel的modify-evaluate-keep/discard核心循环。
---

## 核心逻辑（Autonovel精华）
这是RL里的policy gradient思想：只保留正向reward的trajectory。
- 评分提升 → 同步章节到备份仓库 → git commit保留
- 评分下降 → git reset回滚备份仓库 → 恢复章节到workspace → 重新编辑
- 连续2轮提升幅度<1分 → plateau停止，定稿

## 评分维度（满分100分）
- 爽点密度（25分）：reader-panel 4人格平均爽点分 × 2.5
- 钩子强度（25分）：reader-panel 4人格平均钩子分 × 2.5
- Anti-slop得分（25分）：anti-slop扫描得分（0-100）÷ 4
- 文风一致性（25分）：对照STYLE-GUIDE.md和VOICE-PROFILE.md自评

## 执行步骤

**Step 1：读取本轮各项得分**
从本次对话上下文中读取：
- reader-panel输出的4人格爽点分和钩子分
- anti-slop输出的最终得分（0-100）
- 对照STYLE-GUIDE.md评估文风一致性（0-25）

**Step 2：计算综合评分**
```
爽点密度 = (人格A爽点 + 人格B爽点 + 人格C爽点 + 人格D爽点) / 4 × 2.5
钩子强度 = (人格A钩子 + 人格B钩子 + 人格C钩子 + 人格D钩子) / 4 × 2.5
Anti-slop = anti-slop得分 / 4
文风一致 = 自评0-25分
总分 = 四项之和（满分100）
```

**Step 3：读取上一轮得分**
```bash
python3 -c "
import re
try:
    state = open('./foundation/state.md', encoding='utf-8').read()
    for line in state.split('\n'):
        if '最近一轮评分' in line:
            m = re.search(r'[\d.]+', line)
            if m:
                print(m.group())
                break
    else:
        print('0')
except:
    print('0')
"
```

**Step 4：决策并执行**

**情况A：本轮总分 > 上一轮总分（保留）**

先把workspace的章节同步到备份仓库：
```bash
BACKUP_DIR=~/IronPen
mkdir -p $BACKUP_DIR/chapters/chapter_X
cp ./chapters/chapter_X/第X章.md $BACKUP_DIR/chapters/chapter_X/第X章.md
```

在备份仓库里commit：
```bash
cd ~/IronPen
git add chapters/chapter_X/第X章.md
git commit -m "第X章 第N轮 得分Y"
cd -
```

更新state.md（见Step 5）。

**情况B：本轮总分 < 上一轮总分（回滚）**

回滚备份仓库到上一版本：
```bash
cd ~/IronPen && git reset --hard HEAD~1 && cd -
```

从备份仓库恢复章节文件到workspace：
```bash
cp ~/IronPen/chapters/chapter_X/第X章.md ./chapters/chapter_X/第X章.md
```

重新执行adversarial-edit → reader-panel → anti-slop → 回到Step 1重新评分。

**情况C：连续2轮提升幅度<1分（plateau定稿）**
停止迭代，定稿当前版本，直接进入novel-summarizer。

**Step 5：更新state.md评分历史**
```bash
python3 -c "
import re
new_score = '[新分数]'
state = open('./foundation/state.md', encoding='utf-8').read()
if '最近一轮评分' in state:
    state = re.sub(r'最近一轮评分[：:][^\n]*', f'最近一轮评分：{new_score}', state)
else:
    state += f'\n最近一轮评分：{new_score}\n'
open('./foundation/state.md', 'w', encoding='utf-8').write(state)
print('✅ state.md已更新')
"
```

## 输出格式
```
本轮综合评分：
- 爽点密度：X/25
- 钩子强度：X/25
- Anti-slop：X/25
- 文风一致：X/25
- 总分：X/100

上一轮：X分
决策：✅保留commit / ⚠️回滚重来 / 🏁plateau定稿
```

## 完成后
报告"git-manager完成，本轮X分，决策：[保留/回滚/定稿]，请执行novel-summarizer"