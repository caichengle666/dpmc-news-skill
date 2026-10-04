# CHANGELOG

## v0.3.4（2026-10-02）

- 外部接口实测打通：`GET /api/external/targets` 与 `POST /api/external/news` 均返回 200（`{"success":true,"created":1}`）。鉴权头纠正为 `X-External-Api-Key`（之前误用 `X-API-Key`，后端不认）；connector 已重建，CLI 同步更新（`targets|prices [id]|post|delete`）。教训：以后端官方文档的头名为准，不要猜。

## v0.3.3（2026-10-02）

- Auth 改走 connector：API Key 已入保险库（`custom.dpmc`），调用经 `~/workspace/skills/dpmc/bin/dpmc_news.py`（authd surrogate 自动带 `X-API-Key` 头）；明文 Key 永不进文件。

## v0.3.2（2026-10-02）

- 新增时效性规则：只收 48 小时内新闻；月度/周度数据以最新一期为准；每条必须确认实际发布时间，确认不了的不推。

## v0.3.1（2026-10-02）

- 输出纪律：正式推送的 [事实] 段 100–200 字、[影响] 段 60–120 字，事实必须交代清楚（时间/主体/经过/关键数字），演示用极简列表不作为正式标准。

## v0.3.0（2026-10-02）

- 影响分级由三级改为五级：🔴red / 🟠orange / 🔵blue / 🟢green / ⚪white（烈度递减）；Bark 仍只推红色；`impact` 字段取值同步更新。

## v0.2.2（2026-10-02）

- 多空标签括号由【】改为 ASCII 半角 []：实测【】会被聊天客户端当成引用标记吞掉，导致用户侧标签不可见。

## v0.2.1（2026-10-02）

- 输出格式增加多空标签：每条标题首加【利多/利空/中性】（以塑化价格涨跌为参照），summary 与 Bark 正文同步带标签。

## v0.2.0（2026-10-02）

- 新增品种聚焦：PA/ABS/PC/PP/PE 五类优先（与站内价格监控主力品种对齐），其他品种仅 red 级别才收；SOURCES 新增生意社（100ppi.com）为重点源。

## v0.1.0（2026-10-02）

- 初版。四大板块（石化行业/下游化工/宏观经济/地缘政策）、三级影响分级（红/橙/普通）、红色走 Bark、每日 10–20 条、输出契约（title/summary/source/url/publish_time/impact）、去重（7 天窗口）、进化机制（版本化/logs/月度复盘/用户反馈即规则）。
- 待办：DPMC 后端 `POST /api/news` 上线 + API Key 入保险库后，建每日 8:00 cron 与每月 1 号复盘 cron。

## v0.3.5（2026-10-03）
- 用户反馈"总是找中文媒体"：SOURCES.md 新增来源优先级原则——地缘/原油/美政策/宏观优先美国西方一手（Reuters/Bloomberg/WSJ/NYT/FT/AP），译成中文推送；国内品种价格装置仍以生意社/隆众/卓创为主；重大事件中美交叉验证。

## v0.3.6（2026-10-04）
- 用户要求"skill 更新一下 api"：SKILL.md Output Contract 字段统一为官方版（content/published_at/sentiment/variety/priority/ttl_days/content_hash），停用旧兼容字段 summary/publish_time/impact（publish_time 不落库，实测确认）。

## v0.3.7（2026-10-04）
- 用户要求：Workflow 新增第 8 步"清理旧闻"——推送后检查面板，同一事件出后续/方向打架/叙事反转时删旧闻（隐藏→永久删除），不同市场口径不算矛盾；清理记日志。
