# 中国平台反爬与风控机制 — 知识库

## 核心概念

### IP 信誉体系
- **数据中心 IP（DC IP）**：服务器 IP，腾讯云/阿里云/AWS/GCP 等
  - 特征：ASN 类型为 "HOSTING" 或 "CLOUD"
  - **XHS 对数据中心 IP 有强拦截**（error_code=300012）
  - B站、知乎、微博对数据中心 IP 也有不同级别拦截
- **住宅 IP（Residential）**：家庭网络，真实用户
  - 价格：kuaidaili 住宅代理约 ¥5-15/GB
  - 推荐：SmartProxy、Oxylabs、Luminati（现已改名 Geosurf）
- **中国 ISP 海外节点**：120.235.x.x（广东电信海外转发）
  - 小红书同样拦截

### Captcha 类型（中国平台常见）
| Type | 平台 | 场景 | 绕过方式 |
|------|------|------|----------|
| 124 | 小红书 | complete_qr_login | 真实浏览器 |
| Turnstile | 知乎 | 搜索前 | 住宅 IP / 住宅代理 |
| Geetest | 微博 | 频繁操作 | selenium / 住宅代理 |
| Slider | 通用 | 异常行为 | 指纹浏览器 / 住宅代理 |

### 签名算法
中国平台普遍使用 **X-Bogus / x-s、x-t、x-sg** 等签名：
- **小红书 XHS**：使用 xhshow 库（C++ extension）签名
- **B站**：使用 _blive 参数
- **知乎**：使用 zse-96 参数
- **xhs-cli** 已内置 xhshow 签名

## 各平台风控对比

| 平台 | 登录方式 | IP 要求 | 签名 | cookie 有效期 |
|------|----------|---------|------|--------------|
| 小红书 | QR/captcha | **住宅 IP** | X-Bogus (C++) | 长（但含 web_session） |
| 微博 | 用户名密码 | 住宅或高质量 DC | 无明显签名 | 短（7天） |
| B站 | 用户名密码 | DC 可用 | _blive (JS) | 中等（30天） |
| 知乎 | 用户名密码 | DC 可用 | zse-96 (JS) | 长（永久） |
| 豆瓣 | cookie | DC 可用 | 无 | 极长 |

## 爬虫策略优先级

1. **无需登录的 API**（热榜、搜索建议）→ 最优先尝试
2. **公开 cookie**（B站部分 API、微博部分）
3. **用户名密码 + 签名**（cn-scraper-mcp 模式）
4. **住宅代理 + 真实浏览器**（终极方案）
5. **学术替代**（OpenAlex / arXiv / CrossRef）

## IP 类型检测工具

```bash
# 检测 IP ASN 类型
curl -s "https://ipinfo.io/json" | jq .
# 或
curl -s "https://ipwho.is/json" | jq .

# 数据中心 IP 特征
# "org": "Tencent Cloud Computing..." / "Alibaba Cloud..." / "Amazon AWS..."
# "hostname": "*.compute.amazonaws.com"
```

## 中国代理服务

| 服务 | 类型 | 中国节点 | 价格 | 备注 |
|------|------|----------|------|------|
| 快代理 | 住宅+数据中心 | 有 | ¥2-15/GB | 中国本地公司 |
| 太阳 HTTP | 数据中心 | 有 | ¥1-5/GB | 便宜 |
| SmartProxy | 住宅 | 有 | $7.5/GB | 国际大厂 |
| Oxylabs | 住宅 | 有 | $8/GB | 国际大厂 |
| SOAX | 住宅 | 部分 | $8/GB | 小批量 |

## 小红书风控细节（经验证）

- **QR 创建**：不需要登录，任何 IP 都可以（实测可行）
- **QR 扫码状态查询**：不需要登录，任何 IP 都可以（实测可行）
- **complete_qr_login**：**必须住宅 IP**，否则 captcha type=124
- **login_activate**：可行，但拿到的是**匿名 session**
- **search_notes**：必须 web_session 有效
- **无登录搜索**：XHS 不支持，robots.txt 完全禁止搜索引擎
