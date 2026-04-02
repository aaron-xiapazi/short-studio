# 共享工具层

> Skills 调用的底层能力。统一接口，统一计费，统一管理。

## 设计理念

工具层解决一个问题：**多个 Skill 都需要的能力，不应该重复实现。**

比如"把一个抖音视频链接转成文案"，选题、采集、文案参考都会用到。抽成共享工具，统一维护。

## 架构

```
┌─────────────────────────────────────┐
│  Skills（调用方）                      │
│  topic / script / review / scrape   │
└──────────────┬──────────────────────┘
               │ 调用
┌──────────────▼──────────────────────┐
│  Tools 层（本文件定义）                │
│  douyin-profile / video2script /    │
│  douyin-search / trending           │
└──────────────┬──────────────────────┘
               │ HTTP
┌──────────────▼──────────────────────┐
│  Studio API（远端服务）               │
│  api.short.studio / skill-api       │
│  统一鉴权 / 计费 / 限流               │
└─────────────────────────────────────┘
```

## 调用约定

所有工具通过 `skill-api/` 下的脚本调用，统一接口：

```bash
# 通用格式
python3 skill-api/call.py <tool_name> [参数...]

# 示例
python3 skill-api/call.py douyin-profile --url "https://www.douyin.com/user/xxx"
python3 skill-api/call.py video2script --url "https://www.douyin.com/video/xxx"
python3 skill-api/call.py douyin-search --keyword "AI创业" --count 20
python3 skill-api/call.py trending --platform douyin
```

返回统一 JSON 格式：
```json
{
  "ok": true,
  "data": { ... },
  "usage": { "credits_used": 1, "credits_remaining": 99 }
}
```

错误时：
```json
{
  "ok": false,
  "error": "insufficient_credits",
  "message": "额度不足，请充值：https://short.studio/pay"
}
```

## 鉴权

- 用户首次使用工具时，本地自动生成 UUID 写入 `config/factory.json` 的 `user_id` 字段
- 调用时通过 `X-Studio-Key: <user_id>` 头部鉴权
- 新用户自带试用额度（如 100 次调用）
- 额度用完后返回付款链接，用户通过微信支付绑定账号

## 工具清单

### 1. douyin-profile — 抖音主页采集

抓取指定抖音用户的公开主页信息和近期视频列表。

**用途：** `/studio-scrape` 采集对标账号、`/studio-init` 录入对标账号时预览

**输入：**
```json
{ "url": "https://www.douyin.com/user/MS4wLjABAAAA..." }
```

**输出：**
```json
{
  "nickname": "张三",
  "signature": "...",
  "follower_count": 120000,
  "video_count": 85,
  "recent_videos": [
    {
      "title": "...",
      "url": "...",
      "play_count": 50000,
      "like_count": 3200,
      "duration": 180,
      "publish_time": "2026-03-28"
    }
  ]
}
```

**消耗：** 1 credit/次

---

### 2. video2script — 视频转文案

给定视频链接，提取音频并转写为文字稿。

**用途：** `/studio-scrape` 将对标视频转为文案存入原子库、`/studio-script` 参考优秀文案

**输入：**
```json
{ "url": "https://www.douyin.com/video/xxx" }
```

**输出：**
```json
{
  "title": "视频标题",
  "duration": 245,
  "transcript": "完整文字稿...",
  "sections": [
    { "start": 0, "end": 15, "text": "Hook 部分..." },
    { "start": 15, "end": 60, "text": "背景铺垫..." }
  ]
}
```

**消耗：** 2 credits/次（含 ASR 成本）

---

### 3. douyin-search — 抖音关键词搜索

按关键词搜索抖音视频，返回排序结果。

**用途：** `/studio-topic` 验证选题热度、`/studio-scrape` 发现同领域内容

**输入：**
```json
{ "keyword": "AI创业", "count": 20, "sort": "relevance" }
```

**输出：**
```json
{
  "keyword": "AI创业",
  "total_results": 1200,
  "videos": [
    {
      "title": "...",
      "url": "...",
      "author": "...",
      "play_count": 100000,
      "like_count": 8000,
      "publish_time": "2026-03-30"
    }
  ]
}
```

**消耗：** 1 credit/次

---

### 4. trending — 热点趋势

获取各平台当前热搜/趋势话题。

**用途：** `/studio-topic` 热点捕捉

**输入：**
```json
{ "platform": "douyin" }
```

**输出：**
```json
{
  "platform": "douyin",
  "updated_at": "2026-04-02T10:00:00Z",
  "topics": [
    { "rank": 1, "title": "...", "heat": 9800000, "url": "..." }
  ]
}
```

**消耗：** 1 credit/次

## 离线降级

当 API 不可用时，Skill 应该能降级运行：

- **选题**：无法自动抓热点 → 提示用户手动输入选题方向
- **采集**：无法抓视频 → 提示用户贴入文案文本
- **文案**：不依赖工具，正常运行
- **审稿**：不依赖工具，正常运行

工具层是增强，不是前置依赖。没有工具，工厂照样能跑。

## 后续扩展

| 工具 | 版本 | 说明 |
|------|------|------|
| `tts` | v2.0 | 文字转语音（声音克隆） |
| `video-compose` | v3.0 | 素材拼接 + 字幕 + 画面 |
| `multi-publish` | v4.0 | 多平台自动发布 |
| `analytics` | v4.0 | 各平台数据回收 |
