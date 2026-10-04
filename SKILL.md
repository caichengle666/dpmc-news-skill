---
name: "dpmc-news"
description: "每天为东莞塑胶监控中心（DPMC, dg.021800.xyz）资讯面板采集塑化资讯：筛选、分级（红/橙/普通）、格式化后推送到面板 API；红色级别同步 Bark 推送。触发：每日 8:00 定时任务，或用户说「今日资讯」「推资讯」。"
---

# DPMC 资讯推送

## Purpose

每天为 DPMC 站资讯面板生产一条"值得看"的塑化资讯流：采集 → 筛选 → 按影响分级 → 推送。
红色级别（重大影响）额外走 Bark 第一时间通知用户。本 skill 带自我进化机制（见末尾）。

## Workflow

1. **读上下文**：先读 `FOCUS.md`（当前重点）和 `SOURCES.md`（源清单与健康状态）。
2. **采集**（四大板块，按 FOCUS 权重分配；品种聚焦：PA/ABS/PC/PP/PE 优先，见 `FOCUS.md`）：
- 石化行业：装置检修 / 投产 / 停车、价格异动、库存变化
- 下游化工：开工率、装置动态
- 宏观经济：PMI、CPI/PPI、社融、出口；mceindex.com 月度页（**只取其转述的官方数字**，见 `references/mceindex-boundary.md`）
- 地缘与政策：伊朗局势、美国石油政策（制裁/SPR）、OPEC+ 动向
3. **筛选**：只留"会影响塑化价格"的；丢弃纯企业软文、重复转载、无关内容。全天 10–20 条。
4. **定级**：`red` / `orange` / `blue` / `green` / `white`（烈度 🔴>🟠>🔵>🟢>⚪），标准见 `references/rating-rubric.md`。
5. **去重**：读 `logs/` 近 7 天已推记录，`url` 或（`title`+`source`）重复的不再推。
6. **格式化**：见 Output Contract。
7. **推送面板**：按 `~/workspace/skills/dpmc/SKILL.md` 的「推送资讯」流程操作（CLI：`~/workspace/skills/dpmc/bin/dpmc_news.py post <body.json>`；每条必须自带 `content_hash` 幂等键）。
8. **清理旧闻**：推送后检查面板，把被新资讯取代、矛盾或过时的旧闻清掉（隐藏 → 永久删除，走 dpmc skill 的两步删除流程）：
   - 同一事件出后续：旧的预告/进展让路给最新决议或结果（如 G7 峰会预告 → G7 达成协议）
   - 方向打架：新旧对同一标的判断相反时删旧的（如"远东现货走强" vs "亚洲树脂全线下跌"）
   - 叙事反转：旧资讯的事实基础已被新事件推翻（如"油价单日大涨"后出现大跌）
   - 不同市场/口径的不算矛盾（如国内现货偏空 vs 北美厂商涨价），保留
   - 每次清理在日志里记下删了哪几条 id 和原因
9. **红色告警**：定级 `red` 的条目，逐条走 Bark 推送（见 `references/bark-push.md`）。其余四级只推面板。
10. **记日志**：在 `logs/YYYY-MM-DD.md` 记录采集数 / 过滤数 / 推送数 / 清理数、各源命中情况、红色条目清单。

## Output Contract

面板推送 body（JSON）：

```json
{"items": [{"title": "🔴[利多] 伊朗局势升级，霍尔木兹海峡运输受扰", "content": "[利多]\n[事实]……\n[影响]……", "source": "金联创", "url": "https://...", "published_at": "2026-10-03 08:00:00", "sentiment": "bullish", "variety": "ALL", "priority": 70, "ttl_days": 3, "content_hash": "自生成幂等键"}]}
```

> 字段名必须用官方版（`content` / `published_at` / `sentiment`），旧兼容字段 `summary` / `publish_time` / `impact` 虽被接受但 `published_at` 不会落库（2026-10-03 实测）。

- `title` 首标签：[利多]/[利空]/[中性]（用半角 ASCII 括号；【】会被聊天客户端当引用标记吞掉，禁用），红色再加 🔴 前缀。判定口径：**以塑化价格涨跌为参照**——利多=推动价格上涨（成本上行、供应收缩、需求走强）；利空=压制价格（成本下行、供应宽松、需求走弱）；中性=多空交织、方向不明或纯数据发布
- `content` 固定三段：[利多/利空/中性]+[事实]+[影响]。[事实]段 100–200 字，必须把时间、主体、事件经过、关键数字交代清楚，**不许一句话带过**（演示用的极简列表不算数）；[影响]段 60–120 字，说清向塑化价格传导的链条。可以精简，但事实必须完整
- `url` 必须是真实来源链接，**禁止编造**；搜不到原文的宁可不推
- `published_at` 用 Asia/Shanghai 时间；`sentiment` 取 `bullish` / `bearish` / `neutral`（对应利多/利空/中性）
- Bark：标题「🔴塑化红色预警」，正文首行带[利多/利空/中性]标签，500 字以内

## Auth

- **DPMC API Key**：已存 Secure Vault（connector `custom.dpmc`），调用走 `~/workspace/skills/dpmc/`（见该 skill 的 Tooling/Auth）。CLI：`~/workspace/skills/dpmc/bin/dpmc_news.py`（catalog/targets/prices/post/list/hidden/purge，详见该 skill），Key 经 authd surrogate 交换自动带上（`X-External-Api-Key` 头；后端也支持 `Authorization: Bearer`），**禁止**在聊天/文件/环境变量里放明文。401/403 先确认请求真的带上了 credential（不用 helper 就等于没带），再怀疑 Key 本身；Key 轮换走 Secure Vault（注意：后端“一键轮换”后旧令牌立即失效）。
- **Bark**：走用户现有自建通道（见 `references/bark-push.md`）。

## Operating Rules

1. 每天 10–20 条封顶；红色 0–3 条，没有就不推，不凑数。
2. mceindex 只引用其转述的官方宏观数字，三个自创指数不用（见 `references/mceindex-boundary.md`）。
3. 中文输出；专业名词顺带中文解释。
4. 只读 `FOCUS.md` / `SOURCES.md`，按复盘机制更新它们；不擅自改本文件的契约部分（Workflow/Output Contract/Auth），要改先报用户。
5. **时效性**：只收 48 小时内的新闻（每天 8:00 运行 → 覆盖昨日 8:00 至今日 8:00）。例外：月度宏观数据（PMI/CPI/PPI/社融/出口/mceindex）、周度开工率与库存——以"最新发布的一期"为准，不受 48 小时限制。**每条必须确认实际发布时间并写入 `publish_time`**，确认不了的不推；持续事件引用旧闻做背景时，正文注明原日期，且主体必须是新进展。

## Evolution（进化机制）

- 版本号记在 `CHANGELOG.md`，每次结构性改动 +0.1 并写清原因。
- `logs/` 是进化燃料：来源连续 7 天零命中 → 降权或替换；某板块长期超量 → 收紧标准。
- **每月 1 号复盘任务**：检查源是否失效（RSS 挂了、网站改版）、宏观发布日历有无变化、按日志重写 `FOCUS.md` / `SOURCES.md`。小改动直接生效并记 CHANGELOG；增减板块等结构改动先报用户拍板。
- **用户一句话反馈**（如"这类少推""多盯装置检修"）→ 当天写成规则记入本文件或 `FOCUS.md`，不用等复盘。
- 采集工具、推送 API、mceindex 页面结构任一变化，同步更新本 skill。
