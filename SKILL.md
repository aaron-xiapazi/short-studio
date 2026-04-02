---
name: studio
version: 0.1.0
description: "Short Studio 内容工厂主入口。当用户需要选题、写文案、审稿、采集对标内容，或管理内容生产流程时使用。"
metadata:
  requires:
    bins: ["lark-cli"]
---

# Short Studio — 主路由

本 Skill 是 Short Studio 内容工厂的主入口。**只做意图识别和路由，不执行具体任务。**

触发方式：`/studio`、"内容工厂"、"帮我做内容"

## 路由规则

收到用户请求后，判断意图并路由到对应子 skill：

| 意图关键词 | 路由到 | 说明 |
|-----------|--------|------|
| 初始化、配置、定位、"我是谁" | `skills/init/` | 首次使用或更新定位 |
| 选题、找选题、热点、"今天做什么" | `skills/topic/` | 选题挖掘 |
| 写稿、文案、口播稿、脚本、"帮我写" | `skills/script/` | 口播稿写作 |
| 审稿、改稿、去AI味、润色 | `skills/review/` | 审稿打磨 |
| 采集、对标、抓取、"看看别人" | `skills/scrape/` | 对标账号采集 |
| 发布、推送 | `skills/publish/` | 发布（v2） |

## 首次使用检测

路由前先检查 `config/factory.json` 是否存在：

- **不存在** → 提示用户："工厂还没初始化。先运行 `/studio-init` 完成定位配置。"
- **存在** → 正常路由

## 无明确意图时

如果用户只说了 `/studio` 没有具体指令：

1. 读取 `config/factory.json` 获取当前定位
2. 显示工厂状态概览：
   - 当前定位摘要
   - 选题库待处理数量（如有飞书多维表格配置）
   - 上次操作记录
3. 问："你想做什么？选题 / 写稿 / 采集 / 其他"

## 子 Skill 列表

| Skill | 路径 | 命令 |
|-------|------|------|
| 初始化 | `skills/init/SKILL.md` | `/studio-init` |
| 选题 | `skills/topic/SKILL.md` | `/studio-topic` |
| 文案 | `skills/script/SKILL.md` | `/studio-script` |
| 审稿 | `skills/review/SKILL.md` | `/studio-review` |
| 采集 | `skills/scrape/SKILL.md` | `/studio-scrape` |
| 发布 | `skills/publish/SKILL.md` | `/studio-publish` |
