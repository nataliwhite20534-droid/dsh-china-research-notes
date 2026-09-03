# 内容验证记录

> 验证时间：2026-09-03
> 验证方法：GitHub Contents API 读取 `xhs-research-trail.md`

## 真实文件结构（中文笔记仓库）

```
dsh-china-research-notes/
├── 30-4-role-architecture.md    (9.3 KB)  4 角色架构
├── 31-4-role-workflow.md        (8.4 KB)  4 角色工作流
├── 40-delivery-summary.md       (3.9 KB)  交付总结
├── xhs-research-trail.md        (5.4 KB)  XHS 探索路径（5+ 种）
├── china-platform-ratelimit.md  (3.3 KB)  平台限流规则
├── china-scraper-tools.md       (3.9 KB)  工具评测
├── debugging-archive.md         (4.3 KB)  调试代码片段
├── lessons-learned.md           (3.5 KB)  经验总结
├── cases/                                  实战案例
└── docs/                                  补充文档
```

## xhs-research-trail.md 真实内容摘要

### 5 条已验证路径

| 路径 | 方法 | 结果 |
|------|------|------|
| 1 | 学术绕路（OpenAlex/arXiv） | ❌ 用户要的是 XHS 帖子 |
| 2 | Jina Reader / Bing site:xhs | ❌ XHS robots.txt 禁止 |
| 3 | cn-scraper-mcp 0.5.0 | ⚠️ XHS 需本地 Chrome + cookie |
| 4 | Playwright + 自建爬虫 | ❌ 数据中心 IP 触发风控 |
| 5 | jackwener/xiaohongshu-cli (⭐2545) | ✅ 反向工程 API + 签名算法 |
| 6 | Playwright + Chromium headless | ❌ 未下载 chromium |

### 关键经验

1. **数据中心 IP 必死**（错误码 300012）
2. **web_session 是命脉**（缺它返回 -101）
3. **QR 登录必须真实浏览器**（camoufox headless 在 Windows 不可用）
4. **最稳方案**：`cn-scraper-mcp` 的 `guided_login` + `XiaohongshuEngine.search()`
5. **替代方案**：`xhs_cli` (xiaohongshu-cli) v0.6.4 完整签名算法

## 跨仓联动关系

```
dsh-xhs-collector     →  基于 cn-scraper-mcp（CDP + Cookie 收割）
dsh-4-role-workflow   →  基于 xhs-cli (反向工程 API)
dsh-china-research-notes →  上述两个工具的踩坑经验合集
```

**反爬优先级**（来自 china-platform-ratelimit.md）：

1. 住宅 IP > 签名算法 > 反指纹浏览器 > 数据中心 IP
2. Web 搜索对 XHS 无效（robots.txt 完全禁止）
3. Cookie 有 24 小时窗口期，过期需重新登录

## README 对照结果

**结论：仓库实际是中文研究笔记，含 5+ 个 markdown 文件，**
**不是 DSH 插件（不需 cordis / patch.yml）。**
README 中描述的「XHS 深度研究」「工具链与工作流」
与实际目录结构基本一致。