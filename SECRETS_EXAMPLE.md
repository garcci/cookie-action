# GitHub Secrets 配置示例

这个文件提供了如何在 GitHub 仓库中配置 secrets 的具体示例。

## 访问 Secrets 配置页面

1. 进入你的 GitHub 仓库
2. 点击 "Settings" 选项卡
3. 在左侧菜单中点击 "Secrets and variables" -> "Actions"

## 配置示例

以下是如何配置 secrets 的具体示例：

### 示例 1: 简单的网站会话保持

```
Name: SITE1_ENDPOINT
Value: https://example.com/user/dashboard

Name: SITE1_METHOD
Value: GET

Name: SITE1_HEADERS
Value: {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"}
```

### 示例 2: 需要身份验证的 API

```
Name: SITE2_ENDPOINT
Value: https://api.example.com/auth/refresh

Name: SITE2_METHOD
Value: POST

Name: SITE2_HEADERS
Value: {"Content-Type": "application/json", "Authorization": "Bearer your-actual-token-here"}

Name: SITE2_DATA
Value: {"refresh_token": "your-refresh-token"}
```

### 示例 3: 登录表单提交

```
Name: SITE3_ENDPOINT
Value: https://example.com/login

Name: SITE3_METHOD
Value: POST

Name: SITE3_HEADERS
Value: {"Content-Type": "application/x-www-form-urlencoded"}

Name: SITE3_DATA
Value: username=myusername&password=mypassword&remember=1
```

### 示例 4: 带自定义头部的请求

```
Name: SITE4_ENDPOINT
Value: https://api.example.com/v1/data

Name: SITE4_METHOD
Value: GET

Name: SITE4_HEADERS
Value: {
  "X-API-Key": "your-api-key-here",
  "Accept": "application/json",
  "User-Agent": "MyKeepAliveBot/1.0"
}
```

### 示例 5: 复杂的 POST 请求

```
Name: SITE5_ENDPOINT
Value: https://api.example.com/v1/users/profile

Name: SITE5_METHOD
Value: PUT

Name: SITE5_HEADERS
Value: {
  "Content-Type": "application/json",
  "Authorization": "Bearer your-access-token",
  "X-Request-ID": "unique-request-id"
}

Name: SITE5_DATA
Value: {
  "user_id": "12345",
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

### 示例 6: 使用初始 Cookie 的配置

```
Name: SITE6_ENDPOINT
Value: https://example.com/protected-page

Name: SITE6_METHOD
Value: GET

Name: SITE6_HEADERS
Value: {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"}

Name: SITE6_COOKIES
Value: sessionid=abc123xyz; csrftoken=def456uvw; user_pref=dark_mode
```

## 重要注意事项

1. **安全性**: 不要在代码中硬编码敏感信息，始终使用 secrets
2. **JSON 格式**: 如果你在 HEADER 或 DATA 中使用 JSON，请确保它是有效的 JSON 格式
3. **URL 编码**: 对于表单数据，请确保进行了适当的 URL 编码
4. **令牌轮换**: 定期更新 secrets 中的令牌和密钥
5. **测试**: 在生产环境中使用前，先测试你的配置
6. **Cookie 格式**: Cookie 应该是分号分隔的键值对，例如："sessionid=abc123; csrftoken=def456"

## 如何获取 Cookie

要获取网站的 Cookie，您可以：

1. 在浏览器中打开网站并登录
2. 按 F12 打开开发者工具
3. 转到 "Network"（网络）选项卡
4. 刷新页面或执行相关操作
5. 点击任何一个请求，在 "Headers"（请求头）部分找到 "Cookie" 字段
6. 复制 Cookie 值并将其作为 SITE*_COOKIES secret 添加

## 常见问题

### Q: 如何测试配置是否正确？
A: 你可以手动触发工作流来测试配置是否正确：
1. 进入你的仓库的 "Actions" 选项卡
2. 选择 "Keep Multiple Cookies Alive" 工作流
3. 点击 "Run workflow" 按钮
4. 查看执行日志确认请求是否成功

### Q: 如何知道请求是否成功？
A: 在工作流执行日志中，你会看到类似以下的输出：
```
site1: STATUS: 200
site1: Request completed successfully
```

### Q: 如果站点不需要身份验证怎么办？
A: 只需配置 ENDPOINT 和 METHOD 即可，其他字段可以留空。

### Q: 如何处理 HTTPS 证书问题？
A: 该脚本使用 Node.js 内置的 HTTPS 模块，会自动处理大多数标准证书。如果遇到自签名证书问题，你可能需要修改脚本以忽略证书验证（不推荐用于生产环境）。

### Q: Cookie 续期是如何工作的？
A: 工作流程如下：
1. 如果存在之前保存的 Cookie 文件，则加载这些 Cookie
2. 如果没有保存的 Cookie 但配置了初始 Cookie，则使用初始 Cookie
3. 发送 HTTP 请求到目标站点
4. 如果响应中包含 Set-Cookie 头，则将新 Cookie 保存到文件中
5. 下次执行时使用更新后的 Cookie