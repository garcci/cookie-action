# Multiple Cookies Keep Alive GitHub Action

这个项目包含一个 GitHub Action 工作流，用于定时向多个指定的 API 端点发送请求，以保持各自会话 cookie 的有效性。

## 工作流说明

文件位置: [.github/workflows/keep-cookie-alive.yml](.github/workflows/keep-cookie-alive.yml)

该工作流具有以下功能：

1. 定时执行（默认每30分钟一次）
2. 支持手动触发
3. 自动管理多个站点的 Cookie（保存和重用）
4. 支持直接使用 cURL 命令格式配置
5. 为每个站点分别保存响应内容供后续分析
6. 并行处理多个站点请求

## 配置说明

### 必需的 Secrets 配置

在仓库的 Settings > Secrets and variables > Actions 中添加以下 secrets，可以配置多个站点：

#### 站点 1-10 配置
对于每个站点，只需配置一个 secret：
- `SITE1_CURL` - 站点1的完整 cURL 命令
- `SITE2_CURL` - 站点2的完整 cURL 命令
- ...
- `SITE10_CURL` - 站点10的完整 cURL 命令

注意：
- 只需要配置你想使用的站点，未配置的站点将被自动跳过
- 每个 secret 应包含完整的 cURL 命令

### 配置示例

以下是一些常见的配置示例：

#### 示例1：简单的 GET 请求
```
SITE1_CURL: curl -X GET "https://api.example.com/user/profile" -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)" -H "Accept: application/json"
```

#### 示例2：带认证的 POST 请求
```
SITE2_CURL: curl -X POST "https://api.example.com/auth/refresh" -H "Content-Type: application/json" -H "Authorization: Bearer your-token-here" -d "{\"refresh\": true}"
```

#### 示例3：带表单数据的 POST 请求
```
SITE3_CURL: curl -X POST "https://api.example.com/login" -H "Content-Type: application/x-www-form-urlencoded" -d "username=myuser&password=mypassword"
```

#### 示例4：使用初始 Cookie 的 GET 请求
```
SITE4_CURL: curl -X GET "https://api.example.com/dashboard" -H "X-API-Key: your-api-key" -b "sessionid=abc123; csrftoken=def456"
```

### 定时配置

工作流默认每30分钟执行一次。要修改执行频率，请编辑工作流文件中的 cron 表达式：

```yaml
schedule:
  - cron: '*/30 * * * *'  # 每30分钟执行一次
```

Cron 语法说明：
- `*/30 * * * *` - 每30分钟执行一次
- `0 * * * *` - 每小时执行一次
- `0 0 * * *` - 每天午夜执行一次

参考: [GitHub Actions cron 语法](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#scheduled-events)

## 工作原理

1. 工作流启动时会检查是否存在之前保存的各站点 cookie 文件
2. 对于每个配置的站点，并行执行对应的请求
3. Node.js 脚本会解析您提供的 cURL 命令并提取 URL、方法、头部、Cookie 和数据
4. 如果响应中包含 Set-Cookie 头，则保存到对应站点的 cookie 文件中
5. 下次执行时会重用这些 cookie

## 使用示例

要使用此工作流，请按照以下步骤操作：

1. Fork 此仓库
2. 在仓库设置中添加所需的 secrets（至少配置一个站点的 cURL 命令）
3. （可选）修改工作流文件中的调度频率
4. 工作流将自动开始运行

## 故障排除

如果工作流执行失败，请检查以下内容：

1. 所有必需的 secrets 是否已正确配置
2. cURL 命令是否有效且可执行
3. API 端点是否可访问
4. 查看具体失败任务的日志以获取详细错误信息
5. 确保 secrets 名称完全匹配（区分大小写）