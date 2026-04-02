---
name: studio-review
version: 0.2.0
description: "Short Studio 审稿模块。编导对口播稿进行去AI味、方法论校验、定位一致性检查。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 审稿

审稿不是"润色"，是按检查清单逐项过关。

触发方式：`/studio-review`、"审稿"、"改稿"、"去AI味"

## 审稿流程

### Step 1：读取文稿

从飞书脚本库找到待审的记录，通过文档链接读取正文：

```bash
# 从脚本库读取待审记录
lark-cli base +record-list --as user --base-token <base_token> --table-id <script_table_id>

# 读取文档正文
lark-cli docs +fetch --as user --doc "<doc_url>"
```

### Step 2：去AI味

参考 `知识库/方法论/去AI味检查清单.md`，逐项检查：

**必须去掉的：**
- "值得注意的是"、"总的来说"、"综上所述"
- "不仅...而且..."、"一方面...另一方面..."
- 过度工整的排比句
- "让我们一起来看看"、"接下来"
- 任何听起来像朗读稿/播音腔的表达

**必须加入的：**
- 口语化：把"然而"换成"但是"，"因此"换成"所以"
- 不完美句式：真人说话有停顿、有口头禅、有不完整的句子
- 具体的比喻：用受众熟悉的事物打比方
- 情绪词：真人表达有情绪，不是冷冰冰的陈述

### Step 3：方法论校验

对照 `知识库/方法论/口播稿框架.md` 检查结构完整性：

- [ ] Hook 前3秒有钩子吗？
- [ ] 每30秒有新信息或转折吗？
- [ ] 论据具体吗？有数据/案例/引用吗？
- [ ] 情绪有起伏吗？
- [ ] 结尾有力吗？有金句吗？
- [ ] 画面指引标记了吗？

### Step 4：定位一致性检查

读取定位文档：
- 观点跟账号定位一致吗？
- 用词风格符合 tone 吗？
- 目标受众能听懂吗？

### Step 5：输出审稿报告 + 修改稿

在对话中输出，像编导给意见一样直接说：

```
## 审稿意见

**去AI味：** 改了 X 处，主要是 [具体说明]
**结构：** Hook ✅ / 节奏 ✅ / 论据需要补 [哪里]
**定位一致性：** ✅

### 修改后全文
[完整修改稿]
```

### Step 6：更新飞书

创作者确认后：

```bash
# 更新文档正文
lark-cli docs +update --as user --doc "<doc_url>" --mode overwrite --markdown "<revised_content>"

# 更新脚本库审核意见和状态
lark-cli base +record-upsert --as user \
  --base-token <base_token> --table-id <script_table_id> \
  --record-id <record_id> \
  --json '{"fields": {"状态": "终稿", "审核意见": "审核通过，改了X处AI味"}}'

# 更新选题状态
lark-cli base +record-upsert --as user \
  --base-token <base_token> --table-id <topic_table_id> \
  --record-id <record_id> \
  --json '{"fields": {"状态": "待审"}}'
```

## 权限

| 操作 | scope |
|------|-------|
| 读取/更新文档 | `docx:document` |
| 读写多维表格 | `bitable:app` |
