# 4 角色工作流 — 实战实现

> 配套文档：30-4-role-architecture.md

## 一、PM 调度器（PM Dispatcher）

**文件**：`C:\Users\tseng\.dsh\profiles\web\agents\pm-dispatcher.mjs`

**职责**：
- 接收用户需求
- 拆解为 3 个子任务（Browser / Programmer / Data）
- 调用对应角色执行
- 收集结果并汇报用户

**伪代码**：
```javascript
// PM 调度器
async function dispatch(userRequest) {
  // 1. 任务拆解
  const tasks = await decomposeRequest(userRequest);
  // 2. 评估风险
  const risks = await techLead.evaluate(tasks);
  // 3. 查找工具
  const tools = await needFinder.find(tasks);
  // 4. 并行执行
  const [browserResult, programmerResult, dataResult] = await Promise.all([
    browserUser.run(browserTask, tools),
    programmer.run(programmerTask, tools),
    dataEngineer.run(dataTask, tools)
  ]);
  // 5. 整合
  return await integrate({ browserResult, programmerResult, dataResult });
}
```

## 二、角色 1：浏览器使用者（Browser User）

**文件**：`C:\Users\tseng\.dsh\profiles\web\agents\browser-user.mjs`

**能力**：
- 抓取网页（DSH 插件 + Playwright）
- 应对反爬（登录、代理、指纹）
- 探索新数据源
- 保存原始数据

**示例：抓取 XHS 搜索结果**
```javascript
// 使用 Playwright 抓取需要登录的页面
import { chromium } from "playwright";

async function fetchXHSSearch(keyword, options = {}) {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    userAgent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/<redacted> Safari/537.36",
    locale: "zh-CN"
  });

  // 加载 cookie（如果用户提供了）
  if (options.cookies) {
    await context.addCookies(options.cookies);
  }

  const page = await context.newPage();
  await page.goto(`https://www.xiaohongshu.com/search?keyword=${encodeURIComponent(keyword)}`);

  // 等待结果加载
  await page.waitForSelector(".note-item", { timeout: 10000 });

  // 提取数据
  const results = await page.$$eval(".note-item", items =>
    items.map(item => ({
      title: item.querySelector(".title")?.textContent,
      author: item.querySelector(".author")?.textContent,
      url: item.querySelector("a")?.href,
    }))
  );

  await browser.close();
  return results;
}
```

## 三、角色 2：编程专家（Programmer）

**文件**：`C:\Users\tseng\.dsh\profiles\web\agents\programmer.mjs`

**能力**：
- 安装 DSH 插件（pnpm/npm）
- 安装 Python 包（pip）
- 编写脚本
- 配置工具

**示例：安装并配置 XHS 工具**
```javascript
import { execSync } from "node:child_process";

function installXhsTools() {
  // 1. Python 包
  execSync("pip install xiaohongshu-cli", { stdio: "inherit" });
  execSync("pip install cn-scraper-mcp", { stdio: "inherit" });

  // 2. DSH 插件
  execSync("cd C:\\Users\\tseng\\.dsh\\profiles\\web && pnpm add xiaohongshu-skill", { stdio: "inherit" });
  execSync("cd C:\\Users\\tseng\\.dsh\\profiles\\web && pnpm add my-collection-skills", { stdio: "inherit" });

  // 3. 验证
  const out = execSync("python -c \"import xhs_cli; print('ok')\"").toString();
  console.log("XHS tools:", out);
}
```

## 四、角色 3：数据工程师（Data Engineer）

**文件**：`C:\Users\tseng\.dsh\profiles\web\agents\data-engineer.mjs`

**能力**：
- 解析原始数据（JSON/HTML/PDF）
- 提取符合特征的字段
- 清洗、转换、格式化
- 输出结构化数据集

**示例：从 XHS 原始数据提取 10 条帖子**
```javascript
import fs from "node:fs";

function extractTop10(rawResults, options = {}) {
  // 1. 过滤时间（近 7 天）
  const cutoff = Date.now() - 7 * 24 * 60 * 60 * 1000;
  const recent = rawResults.filter(r => new Date(r.published_at).getTime() > cutoff);

  // 2. 提取必需字段
  const extracted = recent.map(r => ({
    note_id: r.id,
    title: r.title,
    author: r.user?.nickname,
    published_at: new Date(r.published_at * 1000).toISOString(),
    likes: r.liked_count || 0,
    comments: r.comment_count || 0,
    url: `https://www.xiaohongshu.com/explore/${r.id}?xsec_token=${r.xsec_token}`,
  }));

  // 3. 排序：likes + comments 加权
  const sorted = extracted.sort((a, b) => (b.likes * 1 + b.comments * 2) - (a.likes * 1 + a.comments * 2));

  // 4. 取前 N 条
  return sorted.slice(0, options.limit || 10);
}

// 保存为 JSONL
function saveAsJsonL(data, path) {
  const lines = data.map(d => JSON.stringify(d, null, 0)).join("\n");
  fs.writeFileSync(path, lines, "utf8");
}
```

## 五、协同工作流脚本

**文件**：`C:\Users\tseng\.dsh\profiles\web\agents\workflow.mjs`

**调用示例**：
```bash
node workflow.mjs "获取小红书近 7 天关于珠海家教的 10 条帖子"
```

**实现**：
```javascript
// workflow.mjs - PM 调度器
import { browserUser } from "./browser-user.mjs";
import { programmer } from "./programmer.mjs";
import { dataEngineer } from "./data-engineer.mjs";

async function workflow(userRequest) {
  console.log("[PM] 收到需求:", userRequest);

  // 阶段 0: 需求拆解
  const parsed = await parseRequest(userRequest);
  console.log("[PM] 拆解结果:", parsed);

  // 阶段 1: 准备工具
  console.log("[PM] 阶段 1: 让编程专家准备工具");
  const tools = await programmer.prepareTools(parsed);

  // 阶段 2: 浏览器使用者抓取
  console.log("[PM] 阶段 2: 让浏览器使用者抓取数据");
  const raw = await browserUser.fetch(parsed, tools);

  // 阶段 3: 数据工程师提取
  console.log("[PM] 阶段 3: 让数据工程师提取数据");
  const data = await dataEngineer.extract(raw, parsed);

  // 阶段 4: 汇报用户
  console.log("[PM] 阶段 4: 汇报用户");
  const report = {
    summary: `找到 ${data.length} 条帖子`,
    data,
    toolchain: tools.used,
    limitations: raw.limitations || [],
  };
  return report;
}

// 调用
const request = process.argv.slice(2).join(" ");
workflow(request).then(report => {
  console.log("\n[PM] 完成!");
  console.log(JSON.stringify(report, null, 2));
}).catch(err => {
  console.error("[PM] 失败:", err.message);
  process.exit(1);
});
```

## 六、调用现成工具的封装

### 调用 dsh-need-finder
```javascript
async function findTools(need) {
  // 实际：调用 dsh-need-finder 暴露的 tool
  return await dsh.tools.call("plugin_guide", { need });
}
```

### 调用 dsh-tech-lead
```javascript
async function assessRisk(task) {
  return await dsh.tools.call("tech_lead_task_tiering", { task });
}
```

### 调用 dsh-trending-hub
```javascript
async function getTrending(platform, limit = 10) {
  return await dsh.tools.call("trending_get", { platform, limit });
}
```

### 调用 dsh-scrape-webpage
```javascript
async function scrapeWebpage(url) {
  return await dsh.tools.call("webpage_scrape", { url });
}
```

## 七、激活与停用

### 何时激活 4 角色工作流？
1. 用户说："帮我系统化 X 任务"
2. 用户说："给我构建一个工作流"
3. 用户说："成立一个小组"
4. PM 主动判断：当前任务需要多个能力协作

### 何时停用？
1. 用户说："快速" / "简单" / "临时"
2. 任务已经完成
3. 用户中断

## 八、失败处理

### Browser User 失败
1. PM 评估：是否需要换工具？
2. 调用 Programmer 安装替代工具
3. 重试，最多 3 次
4. 失败时直接报告用户

### Programmer 失败
1. 检查网络（pnpm/npm 镜像）
2. 检查权限（管理员模式）
3. 检查 Python 版本（3.14）
4. 报告具体错误给用户

### Data Engineer 失败
1. 检查原始数据格式
2. 重新设计提取规则
3. 手动补充数据
4. 标记为"部分成功"

## 九、用户协助请求模板

当需要用户协助时，PM 应该这样说：

```
【需要您的协助】

任务：[具体任务]
问题：[具体卡点]
请选择：
  A. [选项 1：简单操作]
  B. [选项 2：更深度操作]
  C. [选项 3：跳过此任务]
  D. 其他
```

## 十、当前状态

- [x] 22 个 DSH 插件已装
- [x] Playwright 1.48.0 + Chromium v1140 已装
- [x] jackwener/xiaohongshu-cli 已装（pip）
- [x] cn-scraper-mcp 已装（pip）
- [x] camoufox 已装（pip，仅 Linux）
- [x] 4 角色架构设计完成
- [x] 工作流 SOP 文档完成
- [ ] 实战脚本（agents/）
- [ ] 第一次实战任务（用户 XHS 需求）

下一步：写实际 agent 脚本并跑通。