# 调试记录归档 — 可复用代码片段

## Python / xhs_cli 相关

### 1. 生成 a1 和 webId cookie
```python
import random, time

def gen_a1():
    prefix = ''.join(random.choices('0123456789abcdef', k=24))
    ts = str(int(time.time() * 1000))
    suffix = ''.join(random.choices('0123456789abcdef', k=15))
    return prefix + ts + suffix  # 52 chars

def gen_webid():
    return ''.join(random.choices('0123456789abcdef', k=32))
```

### 2. 生成小红书 QR PNG（正确方式）
```python
import qrcode

img = qrcode.QRCode(
    version=2,
    error_correction=qrcode.constants.ERROR_CORRECT_M,
    box_size=10,
    border=1
)
img.add_data(qr_url)
img.make(fit=True)
img.make_image(fill_color="black", back_color="white").save(path)
# 注意：img.make() 返回 QRCodeImage 不是 PIL Image
# 必须 .make_image() 再 .save()
```

### 3. 读取已保存的 cookie
```python
from xhs_cli.cookies import load_saved_cookies

cookies = load_saved_cookies()
# 返回 dict：{"a1": ..., "webId": ..., "web_session": ..., ...}
# 注意：load_saved_cookies() 会过滤掉 saved_at 字段
```

### 4. 检测 web_session 是否存在
```python
from xhs_cli.cookies import load_saved_cookies

c = load_saved_cookies()
if not c:
    print("No saved cookies")
elif not c.get("web_session"):
    print("No web_session — needs re-login")
else:
    print("web_session:", c["web_session"][:20])
```

### 5. login_activate 获取匿名 session
```python
client = XhsClient(cookies, request_delay=0)
act = client.login_activate()
# 返回：{"user_id": "...", "session": "...", "secure_session": "..."}
# 注意：这是匿名 session，user_id 与扫码账号不同
web_session = client._http.cookies.get("web_session")
```

### 6. 检测 IP 类型（PowerShell）
```powershell
curl -s "https://ipinfo.io/json" | ConvertFrom-Json | Select-Object ip, org, hostname
# 或
(Invoke-WebRequest -Uri "https://ipinfo.io/json" -UseBasicParsing).Content | ConvertFrom-Json
```

### 7. 检查 xhs_cli 安装路径
```bash
python -c "import xhs_cli; print(xhs_cli.__file__)"
python -c "import xhs_cli; from xhs_cli.client import XhsClient; from xhs_cli.cookies import load_saved_cookies"
```

## DSH / Node.js 相关

### 8. 检查 DSH 插件是否安装
```powershell
dsh plugin list
# 或查看
Get-ChildItem "$env:USERPROFILE\.dsh\profiles\web\node_modules" -ErrorAction SilentlyContinue
```

### 9. DSH 环境下 Python 和 Node 的路径差异
- **run_code** 中用 Node.js 的 `child_process`（`exec` / `spawn`）— Windows 原生路径
- **pwsh** 工具中用 PowerShell — 也用 Windows 原生路径
- **Python sys.path** 在 DSH 的 Node.js 子进程和直接 `python` 调用中有差异
- **建议**：始终用 `C:\Python314\python.exe` 的绝对路径避免歧义

### 10. 从 DSH run_code 中调用 Python（正确方式）
```javascript
const { exec } = await import('node:child_process');
const { promisify } = await import('node:util');
const execAsync = promisify(exec);

// 正确
const r = await execAsync('"C:\\Python314\\python.exe" "C:\\path\\to\\script.py" 2>&1', {
  timeout: 30000,
  encoding: 'utf8',
  windowsHide: true,
  maxBuffer: 1024 * 1024 * 5
});

// PowerShell 方式（推荐，更稳定）
const { pwsh } = tools;
const r = await pwsh({ command: 'C:\\Python314\\python.exe "C:\\path\\to\\script.py" 2>&1', timeoutMs: 30000 });
```

### 11. 从 Python 返回 JSON 给 Node（推荐方式）
```python
import json, sys
result = {"items": [...], "total": 100}
print(json.dumps(result))  # 打印到 stdout
sys.exit(0)  # 正常退出
# Node 端：r.stdout.text 就是 JSON 字符串
```

## 常见错误速查

| 错误信息 | 原因 | 解决 |
|---------|------|------|
| "manifest.json is missing" | camoufox headless=True Windows 不支持 | 换 Linux 或用 playwright |
| "Executable doesn't exist" | playwright chromium 未安装 | `playwright install` |
| "IP blocked by XHS (300012)" | 数据中心 IP 被拒 | 换住宅代理 |
| "Captcha required: type=124" | IP 被识别为非人类 | camoufox 真实浏览器 |
| "无登录信息或登录信息为空" | 缺 web_session | 重新扫码登录 |
| "no_viewport_default deadlock" | camoufox 指纹窗口尺寸问题 | 不修改，用默认值 |
| "Virtual display is only supported on Linux" | camoufox Xvfb Windows 不支持 | 用 Windows 原生浏览器 |
