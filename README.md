# dsh-china-research-notes

> 中国互联网平台数据采集的实战经验合集 — 失败案例、风控规则、工具评测。

## 这是什么

在 AI Agent 时代，**自动化数据采集**是核心能力。但中国平台（小红书/知乎/微博/抖音）的反爬机制
比海外平台严格得多。本项目记录了通过真实踩坑、试错、迭代得出的**稳定可复现**的解决方案。

适用对象：
- 在 DSH (DeepSeek Harness) 中工作的 AI Agent
- 需要批量抓取中国平台数据的研究者
- 想要了解中国风控体系的技术人员

## 知识地图

### 1. 小红书 (XHS) 深度研究

- [`xhs-research-trail.md`](xhs-research-trail.md) — 5+ 种尝试方法、失败原因、可用方案
- [`china-platform-ratelimit.md`](china-platform-ratelimit.md) — IP 信誉、captcha、签名算法
- [`china-scraper-tools.md`](china-scraper-tools.md) — 工具对比（cn-scraper-mcp / jackwener / camoufox）
- [`39-cnblogs-liuyanhang-19769674.md`](39-cnblogs-liuyanhang-19769674.md) — Playwright Windows 安装避坑

### 2. 工具链与工作流

- [`debugging-archive.md`](debugging-archive.md) — 可复用代码片段、错误速查
- [`lessons-learned.md`](lessons-learned.md) — 用户反馈 + 工作方法论
- [`00-urls-memory.md`](00-urls-memory.md) — 40+ 个相关网址记忆库

## 核心洞察（速读版）

### XHS 数据采集
1. **数据中心 IP 必死** — 错误码 300012（IP存在风险）
2. **web_session 是命脉** — 缺它返回 -101 无登录信息
3. **完整 QR 登录必须真实浏览器** — camoufox headless 在 Windows 不可用
4. **最稳方案**：`cn-scraper-mcp` 的 `guided_login` + `XiaohongshuEngine.search()`

### 中国平台通用
1. **反爬优先级**：住宅 IP > 签名算法 > 指纹浏览器 > 数据中心 IP
2. **Web 搜索对 XHS 无效** — robots.txt 完全禁止
3. **Cookie 有 24 小时窗口期** — 过期需重新登录

### 工作方法论
1. **先找现成工具**（GitHub / Awesome 列表 / DSH Plugin Market）
2. **5 分钟决策**：IP/网络/工具/数据四个检查点
3. **失败时记录** — 把错误保存为可复用的调试档案
4. **不重复造轮子** — 工具就绪就用工具

## 快速上手

```python
# 推荐方案（cn-scraper-mcp 0.5.0+）
from cn_scraper_mcp.engines.xiaohongshu import XiaohongshuEngine

# 首次：让 cn-scraper-mcp 引导登录
# from cn_scraper_mcp.cookie_harvest import guided_login
# guided_login("xiaohongshu", port=9251, timeout=120)

engine = XiaohongshuEngine()
result = engine.search("你的关键词", limit=10)
for item in result["items"]:
    print(item["title"], "|", item["author"])
```

## 适用工具

| 工具 | 适用场景 | 难度 |
|------|----------|------|
| [cn-scraper-mcp](https://github.com/goesByhc/cn-scraper-mcp) | 全平台通用 | ⭐⭐ |
| [jackwener/xiaohongshu-cli](https://github.com/jackwener/xiaohongshu-cli) | XHS 专用 CLI | ⭐⭐⭐ |
| [camoufox](https://github.com/daijro/camoufox) | Firefox 反指纹（Linux only）| ⭐⭐⭐ |
| [cn-scraper-mcp] guided_login | 引导登录、自动收割 cookie | ⭐ |

## 贡献

欢迎 PR：
- 新的失败案例（附带完整错误信息）
- 新的工具评测
- 新的风控规则发现

## 许可证

MIT — 详见 [LICENSE](LICENSE)