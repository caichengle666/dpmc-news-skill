# Bark 推送规范

沿用用户现有链路（与 LT/狠人视频号 pipeline 同一套）：

- 通道：用户自建 Bark 服务（Cloudflare Workers，`bark.021800.xyz` 自定义域名）
- 方法：**GET**（该服务端 POST 返回 500，不支持 POST）
- 参数：`title`、`body` 必须 `quote(safe='')` 编码；过长时分页（单条 500 字以内）
- 代理偶发 `RemoteDisconnected`，失败重试 + 补发即可
- 本 skill 专用格式：
- `title`：`🔴塑化红色预警`
- `body`：与面板 `summary` 同文（+两段）
- 只推 red 级别；orange/normal 不走 Bark
- 推送成功以服务端返回 code 200 为准，记入当日 log
