---
name: studio-review
version: 0.1.0
description: "Short Studio 审稿模块。对口播稿进行自检、去AI味、方法论校验。当用户需要审稿、改稿、去AI味时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 审稿

审稿不是"润色"，是按检查清单逐项过关。

触发方式：`/studio-review`、"审稿"、"改稿"、"去AI味"

## 审稿流程

### Step 1：读取文稿

```bash
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

读取 `config/factory.json` 中的定位：
- 这篇稿子的观点跟账号定位一致吗？
- 用词风格符合设定的 tone 吗？
- 目标受众能听懂吗？

### Step 5：输出审稿报告 + 修改稿

```
## 审稿报告

### 去AI味
- [x] 已去除 N 处 AI 套话
- [修改细节]

### 结构检查
- [x] Hook ✅ / ❌ [建议]
- [x] 节奏 ✅ / ❌ [建议]
- ...

### 定位一致性
✅ / ❌ [说明]

### 修改后全文
[完整修改稿]
```

### Step 6：更新文档

用户确认后更新飞书文档：

```bash
lark-cli docs +update --as user --doc "<doc_url>" --mode overwrite --markdown "<revised_content>"

lark-cli base +record-update --as user \
  --app-token <topic_base_token> --table-id <topic_table_id> \
  --record-id <record_id> \
  --data '{"fields":{"状态":"已审"}}'
```

## 权限

| 操作 | scope |
|------|-------|
| 读取/更新文档 | `docx:document` |
| 读写多维表格 | `bitable:app` |
