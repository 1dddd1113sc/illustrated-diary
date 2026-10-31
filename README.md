# 爱日记 · Illustrated Diary

<p align="center">
  <b><a href="https://www.loveaiyou.xyz">www.loveaiyou.xyz</a></b> ·
  QQ 群「爱 token」<b>1097902100</b> ·
  <a href="https://1dddd1113sc.github.io/illustrated-diary/">落地页</a>
</p>

> **每个 AI 伴侣都说会记住你。只有我们能证明。**

你把日子写下来，我们把它画成画册——画风统一、主角永远是你。所有内容存在云端，不跟着某台设备走：三年后换手机、换电脑、换平板，翻开还是那本画册，第 1 天和第 100 天的你长得一个样。

---

**说明：代码不开源。** 本仓库用于产品展示、技术写作和公开路线图。参与方式见 [CONTRIBUTING.md](CONTRIBUTING.md)。许可：专有，保留所有权利。

---

## 你会得到什么

**① 记忆存在云端，不跟着设备走**
日记、插画、人物设定全部落在服务端（对象存储 + 数据库），不是存在手机本地缓存的聊天记录。手机丢了、换机了、平板和电脑轮流用——登录同一个账号，看到的是同一本完整的画册。

**② 长期保留，越写越厚**
一条条日子攒成一本可以往回翻的相册。三个月前的画还在，人没变，那天的光线和情绪都在。这是"记得你"和"证明记得你"的区别。

**③ 随时切换设备，不丢一个字**
H5 网页端，手机、平板、电脑同一个账号通用。在地铁上用手机写两句，回家用电脑接着写，配图、风格、历史记录全部同步。

**④ 移动端就能看 AI 小鱼的日记**
AI 伴侣（你可以给它起名叫小鱼）不是你一个人在写——它会自己写日记、自己配插画，把它的那一半日子也放进同一本画册。走路、等车、躺着，掏出手机就能翻它今天写了什么、画了什么。

## AI 伴侣的记忆，问题在哪

AI 伴侣最大的毛病不是遗忘，是它的记忆没有重量。聊天记录里躺着一句"我记得你"，往上翻两屏就没了。但一个被画出来的日子，你会停下来看。

我们给记忆一副身体：

| 别人家的"记忆" | 这里的记忆 |
|---|---|
| 聊天记录里的一行字 | 画册里一页摸得着的画 |
| "我记得你"（嘴上说的） | 第 1 天和第 100 天同一个你（眼睛看到的） |
| 划过去就忘的文字 | 会停下来看的画面 |
| 存在某台手机上 | 存在云端，换设备还在 |

## 它是怎么做的

**1. 一篇日记，拆成 4–8 个分镜。**
市面上的 AI 日记（Day One、Rosebud、Mindsera）最多给整篇配一张装饰图。我们先做事件级切分：把日记拆成一个个具体时刻，每个场景有自己的构图、光线和情绪——像你这一天的电影分镜。

**2. 主角永远不变。**
用你的照片生成角色锚图。"你"在第 1 天和第 100 天是同一个人、同一种画风。这个跨天一致性是所有同类产品都没解决的问题，也是我们的核心壁垒。

**3. 废片不会到你手上。**
生图有约 10% 的废片率。每一张生成后都会过自动一致性校验，不合格的自动重排 prompt 重出——你按一下按钮，拿到的永远是整套。

**4. AI 伴侣也在这本画册里写日记。**
伴侣不是挂在旁边的聊天框。它会读你的图文时间线，自己写日记、自己配插画，它的日记和你的日记共用同一套人物设定，翻开就是两个人在同一本相册里过日子。

**5. 存储与同步是底座，不是附加功能。**
账号体系 + 云端对象存储 + 数据库，写日记、生图任务、插图文件都在服务端。所以"换设备不丢"不是一句承诺，是架构决定的。

## 数字（不吹形容词）

- 一篇 500 字日记 → 6 张插画 → 约 90 秒
- 每篇日记 4–8 个场景，画风与人物跨篇一致
- 废片率约 10%，用户可见废片 0（自动重出）
- 单篇成本按任务记账（token / 张数 / 站点），详细口径见 [docs/architecture.md](docs/architecture.md)

## 截图

| 写 | 今天 | 首页 |
|---|---|---|
| _截图待更新_ | _截图待更新_ | _截图待更新_ |

> 界面正在改版，旧截图已移除，新界面截图到位后填回 `assets/`：
> `write.png` · `today.png` · `home.png` · 首屏动图 `hero.gif`

## 为什么是现在

- 事件级配图：没人做
- 跨天人物一致：没人做
- 日记订阅付费已被验证（Rosebud、Mindsera 约 $13–15/月）
- 窗口期约 12–18 个月

完整分析：[docs/market.md](docs/market.md)

## 路线图

见 [ROADMAP.md](ROADMAP.md)，在 Discussions 里投票。

## 开发日志

代码不开源，但决策全透明：试过什么、什么失败了、花了多少钱，都写在 Discussions 的 Devlog 里。

## 参与

- **Discussions**：功能投票 · 晒画册 · Devlog
- **Issues**：bug 与具体建议（代码不可提供，请勿索取）

---

<details>
<summary><b>English summary</b></summary>

An AI-companion diary whose memory has weight. Every entry becomes 4–8 illustrated
scenes with a permanently consistent protagonist. Everything lives in the cloud, not on
one device: log in on a new phone, tablet or laptop and the whole album is still there.
The AI companion (name it whatever you like) writes and illustrates its own diary pages
inside the same album — readable on mobile anytime.

Source is not open; this repo is for showcase, docs and the public roadmap.

</details>
