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
Name: SITE1_CURL
Value: curl -X GET "https://example.com/user/dashboard" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
```

### 示例 2: 需要身份验证的 API

```
Name: SITE2_CURL
Value: curl -X POST "https://api.example.com/auth/refresh" -H "Content-Type: application/json" -H "Authorization: Bearer your-actual-token-here" -d "{\"refresh_token\": \"your-refresh-token\"}"
```

### 示例 3: 登录表单提交

```
Name: SITE3_CURL
Value: curl -X POST "https://example.com/login" -H "Content-Type: application/x-www-form-urlencoded" -d "username=myusername&password=mypassword&remember=1"
```

### 示例 4: 带自定义头部的请求

```
Name: SITE4_CURL
Value: curl -X GET "https://api.example.com/v1/data" -H "X-API-Key: your-api-key-here" -H "Accept: application/json" -H "User-Agent: MyKeepAliveBot/1.0"
```

### 示例 5: 复杂的 POST 请求

```
Name: SITE5_CURL
Value: curl -X PUT "https://api.example.com/v1/users/profile" -H "Content-Type: application/json" -H "Authorization: Bearer your-access-token" -H "X-Request-ID: unique-request-id" -d "{\"user_id\": \"12345\",\"preferences\": {\"theme\": \"dark\",\"notifications\": true}}"
```

### 示例 6: 使用初始 Cookie 的配置

```
Name: SITE6_CURL
Value: curl -X GET "https://example.com/protected-page" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)" -b "sessionid=abc123xyz; csrftoken=def456uvw; user_pref=dark_mode"
```

## Secrets 配置详解

每个站点支持以下 secrets 配置：

| Secret 名称 | 是否必需 | 说明 |
|------------|---------|------|
| SITE*_CURL | 是 | 完整的 cURL 命令，例如: curl -X GET "https://api.example.com" |

**注意**: 
- `SITE*` 中的 `*` 代表站点编号，从 1 到 10
- `SITE_NAME` 是内部使用的变量，不需要手动配置
- 至少需要配置一个 `SITE*_CURL` 才能启用对应站点的任务

## 重要注意事项

1. **安全性**: 不要在代码中硬编码敏感信息，始终使用 secrets
2. **cURL 命令**: 确保您的 cURL 命令是有效的并且可以正常执行
3. **Cookie 处理**: 系统会自动处理 cookie 的保存和加载，不需要在命令中添加 `-c` 或 `--cookie-jar` 参数
4. **令牌轮换**: 定期更新 secrets 中的令牌和密钥
5. **测试**: 在生产环境中使用前，先测试你的配置

## 如何获取 Cookie

要获取网站的 Cookie，您可以：

1. 在浏览器中打开网站并登录
2. 按 F12 打开开发者工具
3. 转到 "Network"（网络）选项卡
4. 刷新页面或执行相关操作
5. 点击任何一个请求，在 "Headers"（请求头）部分找到 "Cookie" 字段
6. 复制 Cookie 值并在 cURL 命令中使用 `-b` 参数添加

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
A: 只需配置基本的 cURL 命令，例如：`curl -X GET "https://example.com"`

### Q: Cookie 续期是如何工作的？
A: 工作流程如下：
1. 执行您在 secrets 中提供的 cURL 命令
2. 系统自动添加 `-c` 参数将响应中的 Set-Cookie 头保存到文件中
3. 下次执行时，您可以使用 `-b` 参数加载这些 cookie

### Q: SITE_NAME 是什么？需要配置吗？
A: `SITE_NAME` 是内部变量，用于标识不同的站点任务（如 site1、site2 等）。它在工作流内部自动设置，不需要也不应该在 secrets 中手动配置。