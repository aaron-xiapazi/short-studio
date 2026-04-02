---
name: studio-init
version: 0.1.0
description: "Short Studio 工厂初始化。投喂账号定位、个人定位、平台账号、对标账号，完成工厂配置。首次使用或需要更新定位时触发。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# 工厂初始化

首次使用 Short Studio 时，需要告诉工厂"我是谁、做什么、参考谁"。

触发方式：`/studio-init`、"初始化"、"配置工厂"

## 初始化流程

### Step 1：收集基础信息

向用户收集以下信息（逐项引导，不要一次性问完）：

**必填：**
- **账号名称** — 你的内容账号叫什么
- **账号定位** — 这个账号做什么方向（如：AI实战、创业、技术科普）
- **个人定位** — 你是谁，你的差异化是什么（不是"AI博主"，是"每天真的在用AI干活的创业者"）
- **目标受众** — 你的内容给谁看（如：想用AI提效的职场人）
- **内容风格** — 你的表达风格（如：观点鲜明、实战派、不说废话）

**选填：**
- **平台账号** — 抖音/小红书/B站/视频号的主页地址（用于后续采集数据）
- **对标账号** — 你参考谁（账号名 + 平台 + 链接）
- **已有方法论** — 你积累的内容方法论（可以给文件路径或直接说）

### Step 2：生成定位文档

根据收集的信息，生成结构化的定位文档并存入飞书云文档：

```bash
lark-cli docs +create --as user --title "Short Studio 定位文档" --markdown "<positioning_content>"
```

定位文档结构：
```markdown
# [账号名] 定位文档

## 账号定位
[方向描述]

## 个人定位
[差异化描述]

## 目标受众
[受众画像]

## 内容风格
[风格关键词 + 具体说明]

## 对标账号
| 账号 | 平台 | 链接 | 参考点 |
|------|------|------|--------|

## 平台账号
| 平台 | 链接 |
|------|------|

---
*生成时间：YYYY-MM-DD*
*这是活文档，会随着工厂运转持续更新*
```

### Step 3：创建飞书多维表格

创建内容工厂所需的多维表格：

```bash
# 创建选题库
lark-cli base +base-create --as user --name "Studio-选题库"
# 记录 app_token

# 在选题库中创建字段
lark-cli base +field-create --as user --app-token <app_token> --table-id <table_id> --name "角度" --type text
lark-cli base +field-create ... --name "来源" --type text
lark-cli base +field-create ... --name "热度" --type number
lark-cli base +field-create ... --name "状态" --type singleSelect \
  --data '{"property":{"options":[{"name":"待选"},{"name":"已选"},{"name":"写稿中"},{"name":"待审"},{"name":"已审"},{"name":"已发布"}]}}'
lark-cli base +field-create ... --name "文档链接" --type url
lark-cli base +field-create ... --name "备注" --type text
```

### Step 4：写入配置文件

将所有配置写入 `config/factory.json`：

```json
{
  "version": "0.1.0",
  "initialized_at": "2026-04-02T00:00:00Z",
  "creator": {
    "name": "账号名",
    "positioning": "账号定位",
    "personal": "个人定位",
    "audience": "目标受众",
    "tone": "内容风格关键词",
    "platforms": {
      "douyin": "https://...",
      "xiaohongshu": "https://...",
      "bilibili": "https://..."
    }
  },
  "benchmarks": [
    {"name": "对标A", "platform": "抖音", "url": "https://..."}
  ],
  "feishu": {
    "positioning_doc": "飞书定位文档 URL",
    "topic_base_token": "选题库 app_token",
    "topic_table_id": "选题库 table_id"
  }
}
```

### Step 5：确认完成

输出初始化摘要，让用户确认：

```
✅ 工厂初始化完成

定位：[一句话总结]
定位文档：[飞书文档链接]
选题库：[飞书多维表格链接]
对标账号：[数量]个

现在可以开始了：
- /studio-topic  找选题
- /studio-scrape 采集对标账号内容
```

## 更新定位

如果 `config/factory.json` 已存在，进入更新模式：
- 显示当前定位
- 问用户要更新哪部分
- 只更新指定部分，不重新走全流程

## 权限

| 操作 | scope |
|------|-------|
| 创建文档 | `docx:document`, `drive:drive` |
| 创建多维表格 | `bitable:app` |
