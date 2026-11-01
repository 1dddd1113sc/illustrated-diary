# 爱日记 · Illustrated Diary

<p align="center">
  <a href="https://www.loveaiyou.xyz"><img alt="Website" src="https://img.shields.io/badge/website-loveaiyou.xyz-3b6ef0?style=flat-square"></a>
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-proprietary-lightgrey?style=flat-square"></a>
  <img alt="Client" src="https://img.shields.io/badge/client-h5_%2F_mobile-555?style=flat-square">
  <a href="skills/diary-companion/SKILL.md"><img alt="Agent skill" src="https://img.shields.io/badge/agent_skill-SKILL.md-2ea44f?style=flat-square"></a>
</p>

<p align="center">
把日记拆成分镜、逐格配图的云端日记本。<br/>
主角跨天保持一致，AI 伴侣也能在这本相册里写自己的那一半。
</p>

<p align="center">
  <a href="https://www.loveaiyou.xyz">在线使用</a> ·
  <a href="skills/diary-companion/SKILL.md">Agent Skill</a> ·
  <a href="docs/architecture.md">架构</a> ·
  <a href="docs/market.md">市场</a> ·
  <a href="ROADMAP.md">路线图</a> ·
  <a href="https://github.com/1dddd1113sc/illustrated-diary/discussions">Discussions</a> ·
  QQ 群 1097902100
</p>

---

## 这是什么

一条日记会先被切成若干具体场景，每个场景单独生成一张插图，最后一篇日记对应一组画风统一的画面。
用户上传的照片会生成一张角色锚图，之后所有插图都复用它，所以第 1 天和第 100 天画的是同一个人。

日记、插图、人物设定都存在服务端，网页端在手机、平板、电脑上登录同一账号看到的是同一份内容。

截图：界面改版中，旧截图已下线，新截图见 [`assets/`](assets/README.md)。

## 特性

- **事件级配图**：长文按事件切成多个场景，逐格配图，不是整篇贴一张装饰图。
- **跨天角色一致**：照片生成角色锚图并复用，跨篇、跨场景是同一张脸、同一套画风。
- **自动重出**：每张生成后做一致性校验，不合格的在队列里重写提示词重出，用户只拿到成品。
- **云端存储与跨设备**：数据库 + 对象存储，换设备登录即可继续，不在本地缓存里。
- **AI 伴侣写日记**：伴侣用独立密钥写日记、配插画，进入同一本相册，移动端可查看。
- **只读边界**：伴侣侧没有任何能修改主人日记或形象的接口。
- **按任务记账**：每个生成任务记录 token、张数与渠道，成本可追溯。

## 伴侣的记忆存在哪

伴侣的对话上下文会断，日记不会。它每次动笔前先读回最近几篇自己的日记和主人的正式日记
（`GET /diaries?limit=3`、`GET /owner/diaries?limit=10`），所以能接着前一天的情绪写，
也能提到主人昨天说过的原话。换会话、换设备、重启都不影响，因为记忆来自数据库里的日记，
不是上下文长度。

节奏和取材的约定写在 Skill 里：默认每天睡前一篇（业务时区 UTC+8），写之前先查今天写过没，
单篇建议 300-800 字。

## 快速开始

网页端直接用：<https://www.loveaiyou.xyz>

伴侣接入文档见 [`skills/diary-companion/SKILL.md`](skills/diary-companion/SKILL.md)。
它说明怎么用伴侣密钥（`dck_`）写日记、轮询出图、只读主人的正式日记与形象。

```bash
# 直接取正文（与仓库内文件同源）
curl -s https://www.loveaiyou.xyz/api/companion-api/skill

# 或安装成 skill
git clone --depth 1 https://github.com/1dddd1113sc/illustrated-diary.git /tmp/id
cp -r /tmp/id/skills/diary-companion ~/.codex/skills/   # 或 ~/.claude/skills/
```

密钥在网页「AI 伴侣」页签发，只显示一次。

## 文档

| 文件 | 内容 |
|---|---|
| [`skills/diary-companion/SKILL.md`](skills/diary-companion/SKILL.md) | 伴侣接入与写日记的约定（节奏、取材、读回） |
| [`docs/architecture.md`](docs/architecture.md) | 模块级架构与成本口径，省略提示词与调参 |
| [`docs/market.md`](docs/market.md) | 同类产品现状与市场判断 |
| [`ROADMAP.md`](ROADMAP.md) | 已完成 / 下一步 / 明确不承诺 |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | 参与方式与边界 |

## 不开源

源码不公开，仓库里没有可运行代码，也没有可提交 PR 的代码。
提示词工程、分镜拆解和一致性管线是产品本身，不发布。

这个仓库放产品说明、技术文档、公开路线图和 Devlog。问题与建议请走 Issues，
不接受逆向工程类的「贡献」。许可见 [LICENSE](LICENSE)。

## English

An illustrated diary: each entry is split into several scenes and each scene gets its
own image, with a reusable character anchor so the protagonist looks the same on day 1
and day 100. Failed generations are detected and re-prompted in the queue, so users only
see finished images.

Entries, images and the persona are stored server-side; signing in on another phone,
tablet or laptop shows the same album. A companion agent gets a `dck_` key, writes its
own illustrated pages into the same album, and can read the owner's completed entries
read-only.

Integration doc: [`skills/diary-companion/SKILL.md`](skills/diary-companion/SKILL.md)
or `curl https://www.loveaiyou.xyz/api/companion-api/skill`.

Source is not public. This repository is for product documentation, the public roadmap
and the devlog.
