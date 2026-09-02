# DSH Workstation Setup Summary 2026-09-02

## Goal Achieved

Goal: Build a China platform data collection AI workstation (XHS / Bilibili / Zhihu).

**Result: Core system established; some platforms need manual auth**

---

## Delivered

### 1. Installed Plugins (22)

| Plugin | Purpose |
| --- | --- |
| dsh-web-search-pro | Multi-engine search |
| dsh-scrape-webpage | Generic web scraping |
| dsh-trending-hub | B/Douyin/Weibo trending |
| dsh-community-hot | Community hot lists |
| dsh-rss / dsh-rss-digest | RSS subscription + digest |
| dsh-browser | Browser automation |
| dsh-deepread | Deep reading (books/papers) |
| dsh-ai4scholar | Academic search + reference check |
| dsh-knowledge-base | Local knowledge base |
| dsh-bookmarks | Bookmarks |
| dsh-plugin-academic-writing | Academic writing helper |
| dsh-recipe | Workflow recipes |
| dsh-wechat-mp | WeChat OA publishing |
| dsh-need-finder | Need discovery |
| dsh-file-upload | File upload |
| dsh-reference-checker | Reference verification |
| @240xu/dsh-tech-lead | Tech lead agent |
| @a9i5k4/dsh-auto-memory | Auto memory |
| @a9i5k4/dsh-anchored-monitor | Anchored monitor |
| dshmarket | Plugin market |
| my-collection-skills | Bookmark collection skills |

### 2. Agent Scripts (4 files, 13.9 KB)

- `quick-fetch.mjs` - Playwright fast fetch (executablePath fallback)
- `pm-dispatcher.mjs` - 4-role orchestrator (PM / Programmer / Browser / Data)
- `xhs-search.mjs` - Xiaohongshu search (requires web_session cookie)
- `extractor.mjs` - Data cleanup utility

### 3. Notes / Documentation (10 files, 53 KB)

- `30-4-role-architecture.md` - Architecture design
- `31-4-role-workflow.md` - Workflow detailed
- `china-platform-ratelimit.md` - Anti-bot notes
- `china-scraper-tools.md` - Tool chain notes
- `xhs-research-trail.md` - XHS pitfalls
- `00-index.md`, `00-urls-memory.md`, etc.

---

## Test Results

| Tool | Status | Note |
| --- | --- | --- |
| Playwright Chromium 1140 | OK | Local path confirmed |
| quick-fetch.mjs | OK | Baidu Status 200 |
| pm-dispatcher.mjs | OK | Full 3-Phase run |
| xhs_cli (Python) | PARTIAL | Needs correct Python env |
| XHS search | NEEDS COOKIE | web_session required |
| Bilibili trending | OK | via trending-hub or Playwright |
| Academic search | OK | dsh-ai4scholar |

---

## Known Limitations

### XHS - Cookie Missing
**Problem**: web_session cookie not configured, xhs_cli search unavailable.

**Solutions (pick one)**:

1. **Recommended: Export browser cookie**
   - Chrome already logged into XHS
   - Use EditThisCookie extension to export JSON
   - Save to C:\Users\tseng\.xiaohongshu-cli\cookies.json

2. **cn-scraper-mcp guided_login**
   - `npx cn-scraper-mcp guided_login --platform xiaohongshu`

3. **Local xhs_cli login**
   - `pip install xhs-cli`
   - `xhs login --qrcode` (requires display)

### Python Environment
xhs_cli requires a specific Python environment (suggest conda env). The current DSH run environment Python cannot find xhs_cli module.

---

## Daily Usage Commands

```bash
# Quick fetch any page
cd C:\Users\tseng\.dsh\profiles\web\agents
node quick-fetch.mjs https://target.url

# PM orchestrator (analyze -> decompose -> execute -> report)
node pm-dispatcher.mjs "Get 10 XHS posts about Zhuhai tutoring within 7 days"

# XHS search (needs cookie first)
node xhs-search.mjs homework --limit 10

# DSH built-in
dsh web
dsh plugin list
```

---

## Architecture Highlights

**4-Role PM Dispatcher** design:
- **PM**: Receives needs -> platform/data-type/filter detection -> task decomposition
- **Programmer**: Checks tool availability (pip / npm) -> reports gaps
- **Browser User**: Executes data fetch (XHS API / Playwright / RSS)
- **Data Engineer**: Data cleanup, ranking, dedup, formatting

Each step has logging, graceful degradation on failure, final report + improvement suggestions.

---

*by Kimi @ DSH workstation 2026-09-02*