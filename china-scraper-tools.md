# 中国社媒爬虫工具箱

## 2025 年现状推荐

### 小红书（xiaohongshu / RED）

#### 第一选择：jackwener/xiaohongshu-cli
- **GitHub**：https://github.com/jackwener/xiaohongshu-cli
- **安装**：pip install xiaohongshu-cli
- **Star**：2545
- **依赖**：camoufox, browser-cookie3, xhshow
- **能力**：搜索笔记、用户资料、笔记详情、评论
- **登录**：browser-assisted (camoufox) 或 HTTP QR
- **关键文件**：
  - XhsClient (client.py) — 核心 HTTP 客户端
  - qr_login.py — QR 登录流程（含 complete 详解）
  - signing.py — Web API 签名（a1 + x-s + x-sg）
  - cookies.py — Cookie 持久化
  - client_mixins.py — 各 API 端点

#### 第二选择：xhs996/xhs_spider
- **GitHub**：https://github.com/xhs996/xhs_spider
- **Star**：368
- **特点**：完整逆向，包含所有签名算法

### 通用中国社媒

#### cn-scraper-mcp（⭐34，推荐）
- **GitHub**：https://github.com/goesByhc/cn-scraper-mcp
- **安装**：pip install cn-scraper-mcp
- **架构**：CDP + guided_login + 反检测
- **支持的平台**：
  - 微博热搜 OK
  - B站热门 OK
  - B站搜索 FAIL（412）
  - 知乎 FAIL（2026-07 关闭游客搜索）
  - 小红书 FAIL（需本地 Chrome）

### 浏览器自动化

#### camoufox（仅 Linux）
- **GitHub**：https://github.com/daijro/camoufox
- **特点**：Firefox 反指纹，自动随机 UA/Canvas/WebGL/字体
- **Windows 状态**：仅 headless=False（需要显示器）
- **Linux 状态**：headless=True + Xvfb 完全支持
- **在 Windows 的替代**：用 playwright + 手动设置 UA/指纹

#### Playwright
- **安装**：pip install playwright && playwright install
- **Windows headless**：需要先 playwright install 下载 chromium
- **camoufox vs playwright**：
  - camoufox = 反指纹强，开箱即用指纹随机
  - playwright = 通用，需要手动设置指纹参数

### HTTP 客户端（高级）

#### curl_cffi
- **安装**：pip install curl_cffi
- **用途**：TLS 指纹伪装，绕过 JA3 指纹检测
- **适用**：不需要 JS 渲染但需要绕过 TLS 指纹的站点

#### httpx + mitmproxy
- **用途**：调试完整请求/响应
- **注意**：中国平台普遍使用证书锁定（Certificate Pinning），mitmproxy 会阻断

### 学术数据源（替代方案）

| 数据源 | URL | 覆盖 | 反爬 |
|--------|-----|------|------|
| OpenAlex | https://api.openalex.org | 全球学术论文 | 无 |
| arXiv | https://export.arxiv.org/api | cs/物理/数学 | 无 |
| CrossRef | https://api.crossref.org | 学术论文 | 无 |
| CNKI 知网 | — | 中国学术论文 | **极强** |
| Google Scholar | — | 全球学术 | **极强** |

## Cookie 获取对比

| 方案 | 难度 | 可靠性 | 适用平台 |
|------|------|--------|----------|
| browser-cookie3（浏览器劫持） | 中 | 低（版本相关） | 全平台 |
| camoufox（自动化浏览器） | 低 | 高 | 全平台（Linux） |
| Playwright | 低 | 高 | 全平台（需安装） |
| 手动导出（Cookie Editor 插件） | 最低 | 最高 | 全平台 |
| guided_login（MCP 引导） | 低 | 高 | cn-scraper-mcp |

## DSH 内置插件能力速查

| 插件 | 用途 | 中国平台 |
|------|------|----------|
| dsh-trending-hub | 9 平台热搜 | 知乎/微博/B站 OK |
| dsh-rss-digest | RSS 订阅 | 全部 OK |
| dsh-ai4scholar | 学术搜索 | 全部 OK |
| dsh-scrape-webpage | 通用网页 | 小红书 FAIL |
| dsh-web-search-pro | 增强搜索 | 小红书 FAIL |
| dsh-code-search | 代码搜索 | 全部 OK |
| dsh-rag-chat | 本地文档 QA | 全部 OK |

## 路径备忘

- System Python：`C:\Python314\python.exe`
- User site-packages：`C:\Users\tseng\Python\Python314\site-packages`
- xhs_cli cookies：`C:\Users\tseng\.xiaohongshu-cli\cookies.json`
- DSH profile：`C:\Users\tseng\.dsh\profiles\web\`
- 笔记目录：`C:\Users\tseng\.dsh\profiles\web\notes\`
