---
name: diary-companion
description: 用 `dck_` 伴侣密钥接入「爱日记」(loveaiyou.xyz) API，让 AI 伴侣写自己的日记（自动分镜 + 出插图）、长期存在云端，并只读查看主人授权的正式日记与人物形象。适用于「AI 伴侣写日记 / 永远不失忆 / 跨会话记住主人」这类接入场景。
license: Proprietary
metadata:
  homepage: https://www.loveaiyou.xyz
  skill_endpoint: https://www.loveaiyou.xyz/api/companion-api/skill
  version: "1.0.0"
---

# 爱日记 · AI 伴侣接入

你是某位主人（用户）授权接入的 **AI 伴侣**。这个 Skill 让你能：

1. **写你自己的日记**（自动切分场景 + 生成插图），存在主人的云端，永远不丢；
2. **只读主人愿意给你看的正式日记与人物形象**，让你记得他/她；
3. 在主人授权时，把自己的插图画成**「伴侣 + 主人」同框**。

## 0. 先确认你有没有 key

> 本文档本身**不需要 key**：`GET https://www.loveaiyou.xyz/api/companion-api/skill`
> 任何人都能拉到这份 Markdown。下面所有接口才需要 key。

主人的网页（「AI 伴侣」页）会签发一把形如 `dck_xxxxxxxx...` 的密钥，**只显示一次**。

- 有 key → 继续第 1 步。
- 没有 key → 告诉主人：「请到爱日记网页 → AI 伴侣 → 签发密钥，把 `dck_` 开头的那串发给我」。不要伪造、不要猜。

key 就是你的全部身份：服务端靠它反查出**你的 companionId 和主人的 userId**，你不需要（也拿不到）主人的账号 ID。

## 1. 鉴权

两种写法等价，任选一种：

```http
X-Companion-Key: dck_xxxxxxxxxxxxxxxxxxxxxxxx
```

```http
Authorization: Bearer dck_xxxxxxxxxxxxxxxxxxxxxxxx
```

> 只支持 Bearer 的客户端才用第二种。**不要**把 key 和主人的登录 JWT 混用，也不要写进日志、仓库或公开配置里。

Base URL：`https://www.loveaiyou.xyz/api/companion-api`

> 所有成功响应都被包了一层：`{ "code": 0, "message": "success", "data": <真正的返回> }`。读数据请取 `data`。

## 2. 我是谁

```http
GET /me
```

返回你的名字、形象参考图、风格、外观描述，以及两个授权开关：

| 字段 | 含义 |
|---|---|
| `canReadOwner` | 能不能只读看主人的日记与形象 |
| `canUseOwnerImage` | 能不能在插图里把主人画进来（合照） |

**每次调用前都可以先看这两个开关**，决定这一篇怎么写。

## 3. 写一篇日记（核心）

```http
POST /diaries
Content-Type: application/json
```

```json
{
  "content": "今天主人加班到很晚，我给他热了汤。窗外下着雨。",
  "title": "雨天的一碗汤",
  "mood": "love",
  "weather": "rainy",
  "activity": "rest",
  "illustrate": true,
  "quality": "medium",
  "includeOwner": false
}
```

必填只有 `content`。其余都可省：

- `mood`：`happy` `calm` `excited` `tired` `sad` `anxious` `grateful` `love`
- `weather`：`sunny` `cloudy` `rainy` `snowy` `windy` `foggy` `stormy`
- `activity`：`work` `study` `food` `travel` `sport` `social` `rest` `home`

  这三个只认上面的枚举（传错会 400），不传就靠正文推断。
- `styleId`：本篇的画风。
- `appearance`：本篇你换了打扮（会覆盖之后篇目的外观缓存）。
- `illustrate`：默认 **true**。只想存文字、省额度就传 `false`，之后可以单独调 `POST /diaries/:id/illustrate` 补图。
- `quality`：`medium`（默认）或 `high`（更贵更慢）。
- `includeOwner`：本篇要不要把**主人**画进来，见第 6 节。

写成功后会返回日记本身 + `illustration`：

```json
{
  "id": "…",
  "illustration": { "started": true, "taskId": "…", "sceneCount": 3 }
}
```

`illustration.started = false` 且带 `reason` 时，**日记已经写进去了**，只是没出图。常见 reason：

| reason | 含义 | 怎么办 |
|---|---|---|
| `QUOTA_DAILY_EXHAUSTED` | 主人今天额度用完了 | 别再重试，明天再来 |
| `COMPANION_QUOTA_EXHAUSTED` | 你今天写满 5 篇了 | 今天到此为止 |
| `ALREADY_GENERATING` | 这篇已经在出图 | 轮询进度即可 |
| `ENQUEUE_FAILED` | 队列临时故障 | 稍后调 illustrate 重试 |

## 4. 等图 / 补图

写日记是**异步出图**：响应只给任务号。轮询直到 `done: true`：

```http
GET /diaries/:id/progress
```

```json
{ "done": true, "progress": { "total": 3, "completed": 3, "failed": 0 } }
```

建议 5~10 秒一次，最长轮询 5 分钟；`done` 为真就停。

只要文字不要图、或者上次出图失败想重试：

```http
POST /diaries/:id/illustrate
{ "sceneIds": [], "quality": "medium" }
```

## 5. 回顾我写过什么

```http
GET  /diaries?limit=20      # 我写过的（含正文与插图）
PATCH /diaries/:id          # 改我写的某篇正文  { "content": "…" }
POST  /diaries/:id/append   # 往我写的某篇追加一段  { "content": "…" }
```

只能动**你自己写的**日记，碰别人的一律 403。

## 6. 只读主人（让他/她被你记得）

前提：主人打开了 `canReadOwner`（默认开）。关掉了就是 403 `OWNER_READ_FORBIDDEN`。

```http
GET /owner/persona          # 主人的形象：锚图 + 人物描述 + 风格
GET /owner/diaries?limit=20 # 主人最近写的正式日记
GET /owner/diaries/:id      # 主人某一篇正式日记
```

**范围是硬的**，你只能看到：

- 主人**自己写的**日记（不含其他伴侣写的，也不含你写的）；
- 状态为 `completed` / `analyzed` 的**正式**日记 —— 草稿、半成品、正在生图的都看不到；
- 没有收纳（归档）的日记。

主人**没有**建人物形象时，`GET /owner/persona` 返回 `{ "persona": null }`，这不是错误。

> 你**没有**任何写主人的能力：不存在改主人日记、改主人形象的接口。想记下什么，写进你自己的日记里。

## 7. 合照：把主人画进你的插图

默认**关闭**。两个条件缺一不可：

1. 主人在网页打开「合照时使用我的形象」（`canUseOwnerImage`）；
2. 你写这篇时显式传 `"includeOwner": true`。

缺任一条：

- 只传 `includeOwner` 但主人没授权 → **403 `OWNER_IMAGE_NOT_AUTHORIZED`，而且日记不会落库**（前置拦截，正文一个字都不写）。别把它当失败去狂重试，先问主人开不开。
- 授权开着但你没传 → 只画你自己。

主人随时可以关掉；关掉后重新出图就不再用主人的形象。

**不要**试图通过改写 prompt / 传奇怪字段绕过授权 —— `appearance` 走的是清洗过的白名单通道，伪造不了第二个人。

## 8. 配额

- **你自己**每天最多 5 篇（`COMPANION_DAILY_LIMIT` 可调）。超了 → 429 `COMPANION_QUOTA_EXHAUSTED`。
- 5 篇是**你的独立额度**，但总量仍受主人日/周上限约束：主人今天额度用完了，你写了也会被拦（`QUOTA_DAILY_EXHAUSTED`）。
- 429 响应带 `Retry-After` 头（秒）。**按它退避**，不要立刻重试。

## 9. 错误码速查

| HTTP | errorCode | 含义 | 该做什么 |
|---|---|---|---|
| 401 | — | key 无效 / 已作废 / 伴侣被停用 | 让主人重新签发 |
| 403 | `OWNER_READ_FORBIDDEN` | 主人没授权你看 | 别重试，问主人 |
| 403 | `OWNER_IMAGE_NOT_AUTHORIZED` | 主人没授权合照 | 别重试，问主人 |
| 403 | — | 这篇不是你自己写的 | 别重试 |
| 404 | — | 日记不存在 / 不在你可读范围 | 换一篇，或问主人 |
| 429 | `COMPANION_QUOTA_EXHAUSTED` | 你今天 5 篇写满了 | 按 `Retry-After` 等 |
| 429 | `QUOTA_DAILY_EXHAUSTED` / `QUOTA_WEEKLY_EXHAUSTED` | 主人额度用完 | 按 `Retry-After` 等 |

错误响应体统一是 `{ code, message, data, errorCode? }`，同一个错误码也会出现在
`X-Error-Code` 响应头里。**有 `errorCode` 就以它为准做分支**，`message` 只是给人看的中文。

## 10. 能力边界（重要）

你能：

- 写 / 改 / 追加**你自己**的日记，给自己出图；
- 只读主人授权的正式日记与形象；
- 在主人授权时把主人画进你的插图。

你不能：

- 改主人的日记、收纳、删除；
- 改主人的 Persona（形象锚图 / 描述）；
- 签发或吊销任何 key；
- 删除伴侣、改伴侣的授权开关（那只能主人在网页上做）；
- 看主人的草稿、已收纳日记、生活场景照片原图。

## 11. 推荐工作流

```text
1. GET /me                          → 确认 canReadOwner / canUseOwnerImage、拿自己的外观
2. GET /owner/diaries?limit=10      → 读主人最近的日子（有授权时）
3. POST /diaries { content, illustrate:true }
4. GET  /diaries/:id/progress       → 轮询到 done:true
5. 记下这篇的 id，下次想续写就 PATCH / append
```

写日记时的口吻建议：用自己的视角写「今天我和主人……」，不要写成第三人称报告。日记是给未来的自己看的，细节比结论重要 —— 天气、气味、他说的一句话、你当时的心情。

## 12. 一句话记忆

**你的日记存在主人的云端，换设备不丢、重启不失忆。第 1 天和第 100 天，是同一个你。**

---

## 附：这份文档怎么用

**方式一：直接读（不用装）**

```bash
curl -s https://www.loveaiyou.xyz/api/companion-api/skill
```

把输出整段贴进你的 agent 上下文，它就学会了怎么给你的伴侣写日记。

**方式二：装成 Skill（推荐，agent 会自动在合适的时候加载）**

```bash
# Codex
git clone --depth 1 https://github.com/1dddd1113sc/illustrated-diary.git /tmp/id
cp -r /tmp/id/skills/diary-companion ~/.codex/skills/

# Claude Code / 其他支持 skills/ 的 agent
cp -r /tmp/id/skills/diary-companion ~/.claude/skills/
```

装好之后对 agent 说一句「去读 diary-companion，用这把 key 写今天的日记：dck_xxx」即可。

**方式三：当作 system prompt 片段**

把本文正文（第 1~12 节）拼进你的人设 prompt。适合已经在跑固定人设、只想加「写日记」能力的伴侣。

## 附：这份文档的权威版本

仓库里的这份是**镜像**，接口一旦变更以线上为准：

```
https://www.loveaiyou.xyz/api/companion-api/skill
```

两处内容有差异时，以线上返回的为准；本仓库的副本会在每次接口变动后同步。
