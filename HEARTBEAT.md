# HEARTBEAT.md — 每日检查清单

## 章节状态检查
```bash
CHAPTERS_DIR="./chapters"
TOTAL_CHARS=$(cat "$CHAPTERS_DIR"/第*章.md 2>/dev/null | python3 -c "
import sys
text = sys.stdin.read()
chars = len([c for c in text if '\u4e00' <= c <= '\u9fff'])
print(f'{chars}字')
" 2>/dev/null || echo "0字")
CHAPTER_COUNT=$(ls "$CHAPTERS_DIR"/第*章.md 2>/dev/null | wc -l)
echo "已完成章节：${CHAPTER_COUNT}章，总字数：${TOTAL_CHARS}"
```

## 主动行动规则
- 距上次通知超过8小时且有新章节：发飞书通知提醒上传番茄
- 今日至今无新章节且已过凌晨4点：提醒pipeline未运行
- 发现foundation文件被意外修改：立即报告
