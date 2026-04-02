# 共享工具层

> 编导的眼睛和耳朵。6 个工具，覆盖采集、搜索、趋势、评论。

## 设计理念

工具层解决一个问题：**编导需要的能力，不应该重复实现。**

比如"抓评论区"——选题要看、效果追踪要看、粉丝反馈也要看。抽成共享工具，统一维护。

## 架构

```
┌─────────────────────────────────────┐
│  Skills（调用方）                      │
│  topic / script / review / scrape   │
└──────────────┬──────────────────────┘
               │ 调用
┌──────────────▼──────────────────────┐
│  Tools 层（6 个工具）                 │
│  douyin-profile / video2script /    │
│  douyin-search / trending /         │
│  comments / web-search              │
└──────────────┬──────────────────────┘
               │ HTTP
┌──────────────▼──────────────────────┐
│  Studio API（远端服务）               │
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
python3 skill-api/call.py comments --url "https://www.douyin.com/video/xxx" --count 50
python3 skill-api/call.py web-search --query "2026年AI行业融资数据" --count 10
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

- 首次使用自动生成 UUID → 写入 `config/factory.json` 的 `user_id`
- 调用时 `X-Studio-Key: <user_id>` 鉴权
- 新用户自带试用额度
- 额度用完返回付款链接

## 工具清单

### 1. douyin-profile — 抖音主页采集

抓取指定抖音用户的公开主页信息和近期视频列表。

**编导用在哪：** 采集对标账号、初始化时预览对标

**输入：**
```json
{ "url": "https://www.douyin.com/user/MS4wLjABAAAA..." }
```

**输出：**
```json
{
  "nickname": "张三",
  "follower_count": 120000,
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

**消耗：** 1 credit

---

### 2. video2script — 视频转文案

给定视频链接，提取音频并转写为文字稿。

**编导用在哪：** 素材转写存入素材库、写稿时参考优秀文案

**输入：**
```json
{ "url": "https://www.douyin.com/video/xxx" }
```

**输出：**
```json
{
  "title": "视频标题",
  "duration": 245,
  "transcript": "完整文字稿..."
}
```

**消耗：** 2 credits（含 ASR 成本）

---

### 3. douyin-search — 抖音关键词搜索

按关键词搜索抖音视频，返回排序结果。

**编导用在哪：** 验证选题热度、发现同领域内容

**输入：**
```json
{ "keyword": "AI创业", "count": 20, "sort": "relevance" }
```

**输出：**
```json
{
  "keyword": "AI创业",
  "videos": [
    {
      "title": "...",
      "url": "...",
      "author": "...",
      "play_count": 100000,
      "like_count": 8000
    }
  ]
}
```

**消耗：** 1 credit

---

### 4. trending — 热点趋势

获取各平台当前热搜/趋势话题。

**编导用在哪：** 每天早上巡逻抓热点、推荐选题

**输入：**
```json
{ "platform": "douyin" }
```

**输出：**
```json
{
  "platform": "douyin",
  "updated_at": "2026-04-03T10:00:00Z",
  "topics": [
    { "rank": 1, "title": "...", "heat": 9800000, "url": "..." }
  ]
}
```

**消耗：** 1 credit

---

### 5. comments — 抓评论区

抓取指定视频的评论数据。

**编导用在哪：** 粉丝反馈分析、选题灵感、发布效果判断

**输入：**
```json
{ "url": "https://www.douyin.com/video/xxx", "count": 50, "sort": "hot" }
```

**输出：**
```json
{
  "video_title": "...",
  "total_comments": 1200,
  "comments": [
    {
      "user": "用户A",
      "text": "这个观点太对了...",
      "likes": 320,
      "replies": 15,
      "time": "2026-04-02"
    }
  ]
}
```

**消耗：** 1 credit

---

### 6. web-search — 通用搜索

通用网络搜索（博查等），用于背景调研和事实核查。

**编导用在哪：** 写稿时查数据/案例/背景、选题时验证信息、审稿时事实核查

**输入：**
```json
{ "query": "2026年AI行业融资数据", "count": 10 }
```

**输出：**
```json
{
  "query": "2026年AI行业融资数据",
  "results": [
    {
      "title": "...",
      "url": "...",
      "snippet": "...",
      "source": "36kr"
    }
  ]
}
```

**消耗：** 1 credit

## 离线降级

当 API 不可用时，编导降级运行：

- **选题**：无法自动抓热点 → 提示创作者手动输入方向
- **采集**：无法抓视频 → 提示创作者贴入文案文本
- **评论**：无法抓评论 → 提示创作者截图或描述反馈
- **搜索**：无法搜索 → 提示创作者提供背景信息
- **文案/审稿**：不依赖工具，正常运行

工具是编导的增强，不是前置依赖。没有工具，编导照样能陪你创作。

## 后续扩展

| 工具 | 版本 | 说明 |
|------|------|------|
| `tts` | v2.0 | 文字转语音（声音克隆） |
| `video-compose` | v3.0 | 素材拼接 + 字幕 + 画面 |
| `multi-publish` | v4.0 | 多平台自动发布 |
| `analytics` | v4.0 | 各平台数据回收 |
