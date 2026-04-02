---
name: studio-topic
version: 0.2.0
description: "Short Studio 选题模块。编导从热点和对标内容中筛选适合创作者的选题。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 选题

选题不是"找热点"，是**在你的定位下找到适合你讲的热点**。

触发方式：`/studio-topic`、"选题"、"找选题"、"今天做什么"

## 前置检查

1. 读取 `config/factory.json`，获取创作者定位
2. 如果不存在，提示先运行 `/studio-init`

## 选题流程

### Step 1：多渠道抓取

调用共享工具获取数据：

```bash
# 热点趋势
python3 skill-api/call.py trending --platform douyin

# 关键词搜索（围绕定位方向）
python3 skill-api/call.py douyin-search --keyword "<定位关键词>" --count 20

# 通用搜索（补充背景）
python3 skill-api/call.py web-search --query "<定位方向> 最新动态" --count 10
```

如果工具不可用 → 提示创作者手动输入感兴趣的方向或热点。

同时读飞书素材库，看对标账号最近在做什么。

### Step 2：定位匹配筛选

编导内部思考（不输出到表格，在对话中说）：

1. **跟定位有关吗？** — 不相关的直接淘汰
2. **有独特视角吗？** — 能说出别人说不了的东西吗
3. **受众关心吗？** — 目标受众会停下来看吗
4. **有素材支撑吗？** — 能说出干货还是只能泛泛而谈

参考 `知识库/方法论/选题评估矩阵.md` 做内部评估。

### Step 3：输出选题推荐

在对话中推荐 3-5 个选题，直接跟创作者说：

```
## 选题推荐

1. **[标题]**
   角度：[怎么讲]
   为什么适合你：[一句话]
   来源：[哪里看到的]

2. **[标题]**
   ...

3. **[标题]**
   ...

你觉得哪个好？或者有别的想法？
```

**不要输出评分表格。** 直接说推荐理由，像编导跟创作者聊天一样。

### Step 4：写入飞书选题库

创作者选定后，写入飞书多维表格：

```bash
lark-cli base +record-upsert --as user \
  --base-token <base_token> --table-id <topic_table_id> \
  --json '{
    "fields": {
      "标题": "选题标题",
      "角度": "切入角度",
      "状态": "已选",
      "来源": "热点链接或参考来源",
      "备注": ""
    }
  }'
```

### Step 5：参考历史

如果飞书选题库已有记录：
- 检查是否做过类似选题（避免重复）
- 参考已发布选题的方向

如果素材库有相关内容：
- 关联参考素材，写稿时可以用

## 编导主动推选题

编导不只是被动等创作者调用。当定时任务触发时：

1. 调用 trending + web-search 抓当日热点
2. 匹配创作者定位
3. 如果发现好选题 → 通过飞书 IM 推送：

```bash
lark-cli im +messages-send --chat-id <chat_id> --msg-type text \
  --content '{"text":"铁铁，发现一个好选题：[标题]\n角度：[切入点]\n要不要做？"}'
```

## 权限

| 操作 | scope |
|------|-------|
| 读写多维表格 | `bitable:app` |
| 发送消息 | `im:message` |
