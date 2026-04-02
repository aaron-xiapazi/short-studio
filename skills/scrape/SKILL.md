---
name: studio-scrape
version: 0.2.0
description: "Short Studio 采集模块。抓取对标账号的视频内容数据，写入飞书素材库。当用户需要采集对标内容、分析竞品时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 采集

从对标账号采集内容数据，写入飞书素材库。编导的知识积累从这里开始。

触发方式：`/studio-scrape`、"采集"、"看看对标"、"分析竞品"

## 采集流程

### Step 1：读取对标列表

从 `config/factory.json` 的 `benchmarks` 字段获取对标账号列表。

### Step 2：采集内容数据

调用共享工具：

```bash
# 采集对标账号主页
python3 skill-api/call.py douyin-profile --url "<benchmark_url>"

# 对每条视频转写文案
python3 skill-api/call.py video2script --url "<video_url>"
```

如果工具不可用 → 提示创作者手动贴入视频链接或文案文本。

### Step 3：写入飞书素材库

将采集数据写入飞书多维表格的素材库（主数据）：

```bash
lark-cli base +record-upsert --as user \
  --base-token <base_token> --table-id <material_table_id> \
  --json '{
    "fields": {
      "标题": "视频标题",
      "作者": "对标账号名",
      "平台": "抖音",
      "链接": {"text": "原始链接", "link": "https://..."},
      "转写文案": "完整文字稿...",
      "播放量": 50000,
      "点赞量": 3200,
      "采集时间": 1712102400000
    }
  }'
```

**写入前先 `+field-list` 确认字段结构。**

同时写入本地 JSONL 做缓存（完整转写文案可能很长，表格里可截取）：

```bash
# 本地缓存
echo '{"title":"...","author":"...","url":"...","transcript":"完整文案...","play_count":50000,"like_count":3200,"scraped_at":"2026-04-03"}' >> 知识库/原子库/<账号名>.jsonl
```

### Step 4：分析摘要

采集完成后输出分析摘要：
- 该账号最近的内容方向
- 爆款内容的共性（话题、标题风格、时长）
- 跟我们定位的交集点
- 可以借鉴的选题或角度

### Step 5：通知创作者

如果 `factory.json` 中有 `feishu.chat_id`，通过飞书 IM 推送采集结果：

```bash
lark-cli im +messages-send --chat-id <chat_id> --msg-type text \
  --content '{"text":"采集完成！[对标账号] 最近20条内容已入库，发现3个值得参考的选题方向..."}'
```

## 注意

- 只采集公开可见的内容
- 采集频率不要太高，避免触发平台限制
- 本地 JSONL 缓存在 `.gitignore` 中，不推送到 GitHub

## 权限

| 操作 | scope |
|------|-------|
| 写多维表格 | `bitable:app` |
| 发送消息 | `im:message` |
