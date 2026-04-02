---
name: studio-scrape
version: 0.1.0
description: "Short Studio 采集模块。抓取对标账号的视频内容数据，喂入知识库原子库。当用户需要采集对标内容、分析竞品时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 采集

从对标账号采集内容数据，喂入知识库。工厂的知识积累从这里开始。

触发方式：`/studio-scrape`、"采集"、"看看对标"、"分析竞品"

## 采集流程

### Step 1：读取对标列表

从 `config/factory.json` 的 `benchmarks` 字段获取对标账号列表。

### Step 2：采集内容数据

使用 agent-reach 或 web-access skill 抓取对标账号的公开内容：

**抓取字段：**
- 标题
- 简介/描述
- 发布时间
- 播放量/点赞/评论/转发
- 时长
- 话题标签

### Step 3：结构化存储

每个对标账号存一个 JSONL 文件到 `知识库/原子库/{账号名}/`：

```jsonl
{"id":"001","title":"标题","hook":"前几个字","topic":"所属话题","views":50000,"likes":3000,"comments":200,"duration":180,"tags":["AI","效率"],"platform":"抖音","scraped_at":"2026-04-02"}
```

### Step 4：分析摘要

采集完成后输出分析摘要：
- 该账号最近的内容方向
- 爆款内容的共性（话题、标题风格、时长）
- 跟我们定位的交集点
- 可以借鉴的选题或角度

### Step 5：更新统计

更新 `知识库/原子库/_stats.json`：

```json
{
  "last_scraped": "2026-04-02",
  "accounts": {
    "对标A": {"platform": "抖音", "total_atoms": 50, "last_update": "2026-04-02"},
    "对标B": {"platform": "小红书", "total_atoms": 30, "last_update": "2026-04-01"}
  }
}
```

## 注意

- 只采集公开可见的内容
- 采集频率不要太高，避免触发平台限制
- 原子库在 `.gitignore` 中，不推送到 GitHub（含具体数据）

## 权限

无需额外飞书权限。依赖 web-access / agent-reach skill。
