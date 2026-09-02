# 4 角色 AI Agent 工作体系 — 设计文档

> 创建时间：2025-09-01
> 目标：构建一个能进入特殊数据集、提取数据、编写工具、协调用户与团队的智能工作流

## 一、4 个角色总览

### 角色 1：资深浏览器使用者（Browser User）

**职责**：
- 打开和进入"特殊数据集"（GitHub、小红书、知乎、微博、论坛等）
- 寻找并抓取原始数据（笔记、评论、用户信息、文件、代码）
- 应对反爬虫机制（登录、风控、验证码）
- 探索新的数据源和资源

**对应插件和工具**：
- dsh-scrape-webpage（通用网页抓取）
- dsh-web-search-pro（增强搜索）
- dsh-trending-hub（9 平台热榜）
- dsh-rss / dsh-rss-digest（RSS 订阅）
- dsh-community-hot（社区热榜）
- xiaohongshu-skill（XHS 专用）
- my-collection-skills（CookieCloud + Playwright 拉取收藏）
- dsh-ai4scholar（学术搜索）
- Playwright 1.48.0 + Chromium v1140（已装）
- jackwener/xiaohongshu-cli（XHS 备用）
- cn-scraper-mcp（社媒备用）

### 角色 2：编程专家（Programmer）

**职责**：
- 编写工具、脚本、插件
- 吸收并集成第三方工具与框架
- 安装和配置 DSH 插件
- 实现和部署解决方案

**对应插件和工具**：
- DSH 本身（所有 dsh-* 插件）
- dsh-recipe（场景化插件包）
- dsh-plugin-academic-writing（学术写作）
- dsh-need-finder（按需找插件）
- dsh-plugin-audit（插件审计）
- pnpm/npm（包管理）
- Python 3.14 + pip（Python 工具）
- GitHub CLI（代码仓库操作）

### 角色 3：资深数据工程师（Data Engineer）

**职责**：
- 打包和提取"特殊数据集"中符合特征要求的数据
- 数据清洗、转换、格式化
- 存储、检索、索引数据集
- 制定和执行数据提取规则

**对应插件和工具**：
- dsh-knowledge-base（知识库）
- dsh-deepread（深度阅读）
- dsh-bookmarks（书签管理）
- dsh-reference-checker（引用检查）
- dsh-trending-hub（数据聚合）
- Python pandas / numpy / scrapy / beautifulsoup4
- SQLite / DuckDB（嵌入式数据库）
- jq / yq（结构化数据处理）

### 角色 4：资深产品经理（Product Manager）

**职责**：
- 把用户需求转为 3 个角色都能懂的语言
- 实时反馈工作进度
- 在需要时向用户请求协助
- 协调 3 个角色的沟通与协作
- 分配任务、安排优先级、汇报结果

**对应插件和工具**：
- dsh-tech-lead（21 个只读治理工具）
- dsh-need-finder（需求驱动发现）
- dsh-recipe（场景化配置）
- dsh-plugin-audit（插件质量审计）
- @a9i5k4/dsh-auto-memory（自动记忆）
- dsh-project-context（项目上下文）
- dsh-bridge（跨会话通信）
- todo_write / goal tools（任务追踪）

## 二、4 角色工作流 SOP

### 阶段 0：需求接收（PM）
```
用户提出需求
   ↓
PM 用 dsh-need-finder 确认需要哪些插件/工具
   ↓
PM 把需求拆解为 3 个子任务：浏览/编程/数据
   ↓
PM 用 dsh-tech-lead 的 task_tiering 给每个任务分级
```

### 阶段 1：浏览器使用者（Browser User）行动
```
PM 派出任务："用浏览器去 X 网站找 Y 数据"
   ↓
Browser User 评估反爬难度：
   - 简单：直接 dsh-scrape-webpage
   - 中等：dsh-web-search-pro 聚合搜索
   - 困难：Playwright 真实浏览器 + 反指纹
   ↓
抓取到原始数据 → 暂存到 .dsh/profiles/web/cache/
   ↓
汇报：原始数据位置 + 抓取方法 + 遇到的限制
```

### 阶段 2：编程专家（Programmer）介入
```
PM 评估：是否需要新工具？
   ↓
Programmer 决定：
   - 不需要：直接进入阶段 3
   - 需要 npm 工具：pnpm add <pkg>
   - 需要 Python 工具：pip install <pkg>
   - 需要自定义脚本：编写 Node/Python 脚本
   ↓
Programmer 用 dsh-recipe 打包场景
   ↓
汇报：新工具 + 安装方法 + 使用方法
```

### 阶段 3：数据工程师（Data Engineer）提取
```
PM 派出任务："从原始数据中提取符合 X 特征的数据"
   ↓
Data Engineer 设计提取规则：
   - 数据源类型（JSON / HTML / PDF / Markdown）
   - 提取字段（标题、作者、日期、内容）
   - 过滤条件（时间范围、关键词、分类）
   ↓
Data Engineer 用 Python/Pandas 提取 → 输出 CSV/JSONL
   ↓
存储到 dsh-knowledge-base 供检索
   ↓
汇报：提取的数据集 + 元数据 + 质量评估
```

### 阶段 4：PM 反馈与协调
```
PM 整合 3 个角色的输出：
   - 工具列表（来自 Programmer）
   - 原始数据（来自 Browser User）
   - 提取数据（来自 Data Engineer）
   ↓
PM 向用户汇报：
   - 任务完成度
   - 数据摘要
   - 下一步建议
   ↓
PM 用 dsh-auto-memory 记录工作流经验
```

## 三、实际工作实例：获取小红书"珠海家教"10 条帖子

### 用户需求
"获取小红书近 7 天关于珠海家教的 10 条帖子，研究辅导行业与 K12 教育"

### PM 阶段
```
1. PM 用 dsh-need-finder 搜索：
   "Need: extract 10 recent XHS posts about 珠海家教"
   → 找到 xiaohongshu-skill, my-collection-skills, jackwener/xiaohongshu-cli

2. PM 拆解任务：
   - Browser User: 决定登录方式（XHS QR）
   - Programmer: 评估工具（已有 xhs-cli）
   - Data Engineer: 设计提取规则（标题、作者、日期、点赞、评论）

3. PM 用 dsh-tech-lead 评估风险：
   - 高风险：数据中心 IP
   - 中风险：Cookie 保存
   - 低风险：JSON 解析
```

### Browser User 阶段
```
1. 尝试 dsh-scrape-webpage：
   → 失败（XHS robots.txt 禁止）
2. 尝试 dsh-web-search-pro：
   → 失败（XHS 不被搜索引擎索引）
3. 尝试 xiaohongshu-skill：
   → 需要登录态
4. 尝试 my-collection-skills：
   → 需要 Chrome cookie
5. 决定：用户必须在本地 Chrome 登录，导出 cookies
6. 报告 PM：需要用户协助
```

### PM 协调用户
```
PM 报告：
"在数据中心 IP 环境下无法自动登录 XHS。
 请您在本地 Chrome 登录 XHS 后，运行以下命令导出 cookie：
   python -c "import browser_cookie3; ..."
 或者：
   1. 安装 xiaohongshu-cli: pip install xiaohongshu-cli
   2. 在有显示器的环境运行：xhs login --qrcode
   3. 把 cookies.json 传到当前机器"
```

### Programmer 阶段
```
1. 检查已装工具：
   - xiaohongshu-skill 已装
   - jackwener/xiaohongshu-cli 已装（pip）
   - camoufox 装但 Windows headless 失败
2. 决定：用 jackwener/xiaohongshu-cli
3. 写 cookie 加载脚本：xhs-search.py
4. 报告：脚本就绪，等待 cookie
```

### Data Engineer 阶段
```
1. 定义数据 schema：
   {
     note_id, title, author, user_id,
     published_at, likes, comments,
     url, xsec_token
   }
2. 设计去重规则：按 note_id 去重
3. 过滤规则：published_at >= today - 7 days
4. 输出格式：JSONL（每行一条）
5. 报告：规则设计完成
```

### PM 收尾
```
1. PM 等待用户输入 cookies.json
2. 触发完整搜索流程：xhs-search.py 珠海家教
3. 整合 3 个角色的输出
4. 报告用户：
   - 找到 X 条帖子
   - 数据摘要
   - 关键洞察（基于帖子内容）
5. 用 dsh-auto-memory 记录经验
```

## 四、协同工具链（工具间相互调用）

### 协调层（PM）
```
dsh-tech-lead → 任务治理
dsh-need-finder → 工具发现
dsh-recipe → 场景打包
dsh-auto-memory → 经验积累
```

### 浏览器层（Browser User）
```
Playwright 1.48.0
   ↓ 控制
Chromium v1140（已装）
   ↓ 抓取
XHS / GitHub / 各平台 →
   ↓ 原始数据
cache/ → knowledge-base
```

### 编程层（Programmer）
```
pnpm/npm/pip
   ↓ 安装
dsh-* 插件 / Python 包
   ↓ 协调
Browser + Data 层
```

### 数据层（Data Engineer）
```
knowledge-base + deepread + bookmarks
   ↓ 索引
reference-checker
   ↓ 验证
输出数据集（CSV/JSONL/MD）
```

## 五、激活条件

### PM 何时激活 4 角色工作流？
- 用户提出"获取某平台某类数据"
- 用户提出"构建某类工作流"
- 用户提出"系统化某类任务"

### 何时不激活？
- 简单问题（已装工具直接可用）
- 一次性任务（不值得系统化）
- 用户明确说"快速"或"简单"

## 六、相关已装插件（22 个）

### 浏览器相关（5）
- dsh-scrape-webpage（通用抓取）
- dsh-web-search-pro（增强搜索）
- dsh-trending-hub（9 平台热榜）
- dsh-rss-digest（RSS 聚合）
- dsh-rss（RSS 订阅）

### 学术研究（2）
- dsh-ai4scholar
- dsh-plugin-academic-writing

### 数据/知识（5）
- dsh-knowledge-base
- dsh-deepread
- dsh-bookmarks
- dsh-reference-checker
- dsh-community-hot

### 工具集（3）
- dsh-need-finder（找插件）
- dsh-recipe（场景打包）
- dsh-plugin-academic-writing（写作）
- @240xu/dsh-tech-lead（治理）

### 记忆/上下文（2）
- @a9i5k4/dsh-auto-memory
- @a9i5k4/dsh-anchored-monitor

### 专用工具（4）
- xiaohongshu-skill（XHS）
- my-collection-skills（收藏）
- dsh-wechat-mp（微信）
- dsh-file-upload（文件）

### 市场（1）
- dshmarket

### 外部 Python 工具
- xiaohongshu-cli (pip install)
- cn-scraper-mcp (pip install)
- camoufox (pip install, 仅 Linux)
- playwright (npm install -g 1.48.0 + v1140)