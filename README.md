# dpmc-news-skill

DPMC（东莞塑胶监控中心）塑化资讯采集推送 skill：每天采集塑化资讯 → 筛选 → 按影响分级 → 推送到站内资讯面板；红色级别同步 Bark 推送。

入口文档：[SKILL.md](SKILL.md)

- `FOCUS.md` — 当前采集重点（品种、板块）
- `SOURCES.md` — 采集源清单与健康状态
- `references/` — 分级标准、mceindex 边界、Bark 推送说明
- `logs/` — 每日运行日志
- `CHANGELOG.md` — 版本变更记录

配套：资讯经 DPMC 外部 API（`POST /api/external/news`）写入，密钥走 Secure Vault，不在本仓库存放。
