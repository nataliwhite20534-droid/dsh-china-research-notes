ï»¿# dsh-china-research-notes

> ä¸­å½äºèç½å¹³å°æ°æ®ééçå®æç»éªåé â å¤±è´¥æ¡ä¾ãé£æ§è§åãå·¥å·è¯æµã

## è¿æ¯ä»ä¹

å¨ AI Agent æ¶ä»£ï¼**èªå¨åæ°æ®éé**æ¯æ ¸å¿è½åãä½ä¸­å½å¹³å°ï¼å°çº¢ä¹¦/ç¥ä¹/å¾®å/æé³ï¼çåç¬æºå¶
æ¯æµ·å¤å¹³å°ä¸¥æ ¼å¾å¤ãæ¬é¡¹ç®è®°å½äºéè¿çå®è¸©åãè¯éãè¿­ä»£å¾åºç**ç¨³å®å¯å¤ç°**çè§£å³æ¹æ¡ã

éç¨å¯¹è±¡ï¼
- å¨ DSH (DeepSeek Harness) ä¸­å·¥ä½ç AI Agent
- éè¦æ¹éæåä¸­å½å¹³å°æ°æ®çç ç©¶è
- æ³è¦äºè§£ä¸­å½é£æ§ä½ç³»çææ¯äººå

## ç¥è¯å°å¾

### 1. å°çº¢ä¹¦ (XHS) æ·±åº¦ç ç©¶

- [`xhs-research-trail.md`](xhs-research-trail.md) â 5+ ç§å°è¯æ¹æ³ãå¤±è´¥åå ãå¯ç¨æ¹æ¡
- [`china-platform-ratelimit.md`](china-platform-ratelimit.md) â IP ä¿¡èªãcaptchaãç­¾åç®æ³
- [`china-scraper-tools.md`](china-scraper-tools.md) â å·¥å·å¯¹æ¯ï¼cn-scraper-mcp / jackwener / camoufoxï¼
- [`39-cnblogs-liuyanhang-19769674.md`](39-cnblogs-liuyanhang-19769674.md) â Playwright Windows å®è£é¿å

### 2. å·¥å·é¾ä¸å·¥ä½æµ

- [`debugging-archive.md`](debugging-archive.md) â å¯å¤ç¨ä»£ç çæ®µãéè¯¯éæ¥
- [`lessons-learned.md`](lessons-learned.md) â ç¨æ·åé¦ + å·¥ä½æ¹æ³è®º
- [`00-urls-memory.md`](00-urls-memory.md) â 40+ ä¸ªç¸å³ç½åè®°å¿åº

## æ ¸å¿æ´å¯ï¼éè¯»çï¼

### XHS æ°æ®éé
1. **æ°æ®ä¸­å¿ IP å¿æ­»** â éè¯¯ç  300012ï¼IPå­å¨é£é©ï¼
2. **web_session æ¯å½è** â ç¼ºå®è¿å -101 æ ç»å½ä¿¡æ¯
3. **å®æ´ QR ç»å½å¿é¡»çå®æµè§å¨** â camoufox headless å¨ Windows ä¸å¯ç¨
4. **æç¨³æ¹æ¡**ï¼`cn-scraper-mcp` ç `guided_login` + `XiaohongshuEngine.search()`

### ä¸­å½å¹³å°éç¨
1. **åç¬ä¼åçº§**ï¼ä½å® IP > ç­¾åç®æ³ > æçº¹æµè§å¨ > æ°æ®ä¸­å¿ IP
2. **Web æç´¢å¯¹ XHS æ æ** â robots.txt å®å¨ç¦æ­¢
3. **Cookie æ 24 å°æ¶çªå£æ** â è¿æééæ°ç»å½

### å·¥ä½æ¹æ³è®º
1. **åæ¾ç°æå·¥å·**ï¼GitHub / Awesome åè¡¨ / DSH Plugin Marketï¼
2. **5 åéå³ç­**ï¼IP/ç½ç»/å·¥å·/æ°æ®åä¸ªæ£æ¥ç¹
3. **å¤±è´¥æ¶è®°å½** â æéè¯¯ä¿å­ä¸ºå¯å¤ç¨çè°è¯æ¡£æ¡
4. **ä¸éå¤é è½®å­** â å·¥å·å°±ç»ªå°±ç¨å·¥å·

## å¿«éä¸æ

```python
# æ¨èæ¹æ¡ï¼cn-scraper-mcp 0.5.0+ï¼
from cn_scraper_mcp.engines.xiaohongshu import XiaohongshuEngine

# é¦æ¬¡ï¼è®© cn-scraper-mcp å¼å¯¼ç»å½
# from cn_scraper_mcp.cookie_harvest import guided_login
# guided_login("xiaohongshu", port=9251, timeout=120)

engine = XiaohongshuEngine()
result = engine.search("ä½ çå³é®è¯", limit=10)
for item in result["items"]:
    print(item["title"], "|", item["author"])
```

## éç¨å·¥å·

| å·¥å· | éç¨åºæ¯ | é¾åº¦ |
|------|----------|------|
| [cn-scraper-mcp](https://github.com/goesByhc/cn-scraper-mcp) | å¨å¹³å°éç¨ | â­â­ |
| [jackwener/xiaohongshu-cli](https://github.com/jackwener/xiaohongshu-cli) | XHS ä¸ç¨ CLI | â­â­â­ |
| [camoufox](https://github.com/daijro/camoufox) | Firefox åæçº¹ï¼Linux onlyï¼| â­â­â­ |
| [cn-scraper-mcp] guided_login | å¼å¯¼ç»å½ãèªå¨æ¶å² cookie | â­ |

## è´¡ç®

æ¬¢è¿ PRï¼
- æ°çå¤±è´¥æ¡ä¾ï¼éå¸¦å®æ´éè¯¯ä¿¡æ¯ï¼
- æ°çå·¥å·è¯æµ
- æ°çé£æ§è§ååç°

## è®¸å¯è¯

MIT â è¯¦è§ [LICENSE](LICENSE)\n

---

## ð DSH çææå

æ¬é¡¹ç®æ¯ **DSH (DeepSeek Harness)** çæçä¸åï¼åç³»åè¿æï¼

- ð [`dsh-moe-plugin`](https://github.com/nataliwhite20534-droid/dsh-moe-plugin) â èå±æ§ Persona ç³»ç»ï¼10 ç§é¢è®¾å¡çï¼
- ð [`dsh-xhs-collector`](https://github.com/nataliwhite20534-droid/dsh-xhs-collector) â å°çº¢ä¹¦æ¹éæåå·¥å·
- âï¸ [`dsh-4-role-workflow`](https://github.com/nataliwhite20534-droid/dsh-4-role-workflow) â 4 è§è² Agent å·¥ä½æµ

> æ¬¢è¿ Star / Fork / Issueï¼æ³åä¸å¼åï¼Fork åæ PR å³å¯ã

## ð ç¸å³é¾æ¥

- [DSH (DeepSeek Harness) ä¸»ä»](https://github.com/deepseek-ai/deepseek-harness)

---

## 🌏 DSH 生态成员

本项目是 **DSH (DeepSeek Harness)** 生态的一员，同系列还有：

- 🎀 `dsh-moe-plugin` — 萌属性 Persona 系统（10 种预设卡片）
- 📕 `dsh-xhs-collector` — 小红书批量抓取工具
- ⚙️ `dsh-4-role-workflow` — 4 角色 Agent 工作流

> 欢迎 Star / Fork / Issue！想参与开发？Fork 后提 PR 即可。

## 🔗 相关链接

- [DSH (DeepSeek Harness) 主仓](https://github.com/deepseek-ai/deepseek-harness)
