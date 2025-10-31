# Multiple Cookies Keep Alive GitHub Action

这个项目包含一个 GitHub Action 工作流，用于定时向多个指定的 API 端点发送请求，以保持各自会话 cookie 的有效性。

## 工作流说明

文件位置: [.github/workflows/keep-cookie-alive.yml](.github/workflows/keep-cookie-alive.yml)

该工作流具有以下功能：

1. 定时执行（默认每30分钟一次）
2. 支持手动触发
3. 自动管理多个站点的 Cookie（保存和重用）
4. 支持自定义请求方法（GET、POST等）
5. 支持自定义请求头
6. 为每个站点分别保存响应内容供后续分析
7. 并行处理多个站点请求
8. 支持手动设置初始 Cookie

## 配置说明

### 必需的 Secrets 配置

在仓库的 Settings > Secrets and variables > Actions 中添加以下 secrets，可以配置多个站点：

#### 站点 1-10 配置
1. `SITE1_ENDPOINT` - 站点1的目标 API URL
2. `SITE1_METHOD` - 站点1的请求方法（可选，默认为 GET）
3. `SITE1_HEADERS` - 站点1的自定义请求头（可选，JSON 格式）
4. `SITE1_DATA` - 站点1的 POST 请求数据（可选）
5. `SITE1_COOKIES` - 站点1的初始 Cookie（可选）

对于站点2到站点10，使用类似的命名模式：
- `SITE2_ENDPOINT` 到 `SITE10_ENDPOINT`
- `SITE2_METHOD` 到 `SITE10_METHOD`
- `SITE2_HEADERS` 到 `SITE10_HEADERS`
- `SITE2_DATA` 到 `SITE10_DATA`
- `SITE2_COOKIES` 到 `SITE10_COOKIES`

注意：只需要配置你想使用的站点，未配置的站点将被自动跳过。

### 配置示例

以下是一些常见的配置示例：

#### 示例1：简单的 GET 请求
```
SITE1_ENDPOINT: https://api.example.com/user/profile
SITE1_METHOD: GET
SITE1_HEADERS: {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)", "Accept": "application/json"}
```

#### 示例2：带认证的 POST 请求
```
SITE2_ENDPOINT: https://api.example.com/auth/refresh
SITE2_METHOD: POST
SITE2_HEADERS: {"Content-Type": "application/json", "Authorization": "Bearer your-token-here"}
SITE2_DATA: {"refresh": true}
```

#### 示例3：带表单数据的 POST 请求
```
SITE3_ENDPOINT: https://api.example.com/login
SITE3_METHOD: POST
SITE3_HEADERS: {"Content-Type": "application/x-www-form-urlencoded"}
SITE3_DATA: "username=myuser&password=mypassword"
```

#### 示例4：使用自定义头部和初始 Cookie 的 GET 请求
```
SITE4_ENDPOINT: https://api.example.com/dashboard
SITE4_METHOD: GET
SITE4_HEADERS: {"X-API-Key": "your-api-key", "User-Agent": "MyApp/1.0"}
SITE4_COOKIES: "sessionid=abc123; csrftoken=def456"
```

更详细的配置示例请查看 [SECRETS_EXAMPLE.md](SECRETS_EXAMPLE.md) 文件。

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
2. 如果没有保存的 cookie 文件但配置了初始 cookie，则使用初始 cookie
3. 对于每个配置的站点，并行发送 HTTP 请求到指定的 API 端点
4. 如果响应中包含 Set-Cookie 头，则保存到对应站点的 cookie 文件中
5. 下次执行时会重用这些 cookie

## 使用示例

要使用此工作流，请按照以下步骤操作：

1. Fork 此仓库
2. 在仓库设置中添加所需的 secrets（至少配置一个站点）
3. （可选）修改工作流文件中的调度频率
4. 工作流将自动开始运行

## 扩展配置

如果需要添加更多站点，可以按照以下步骤操作：

1. 编辑 [.github/workflows/keep-cookie-alive.yml](.github/workflows/keep-cookie-alive.yml) 文件
2. 在 `matrix.site` 部分添加新的站点配置项
3. 添加对应的执行步骤并引用相应的 secrets
4. 在仓库 secrets 中添加新站点对应的 secrets

例如添加第11个站点：
```yaml
matrix:
  site: 
    - { name: "site1" }
    # ... 其他站点
    - { name: "site10" }
    - { name: "site11" }  # 新增站点
```

以及对应的执行步骤：
```yaml
- name: Execute Keep Alive Request for Site 11
  if: matrix.site.name == 'site11' && secrets.SITE11_ENDPOINT != ''
  env:
    SITE_NAME: site11
    API_ENDPOINT: ${{ secrets.SITE11_ENDPOINT }}
    REQUEST_METHOD: ${{ secrets.SITE11_METHOD || 'GET' }}
    CUSTOM_HEADERS: ${{ secrets.SITE11_HEADERS }}
    POST_DATA: ${{ secrets.SITE11_DATA }}
    INITIAL_COOKIES: ${{ secrets.SITE11_COOKIES }}
  run: |
    node keep-alive.js
```

## 故障排除

如果工作流执行失败，请检查以下内容：

1. 所有必需的 secrets 是否已正确配置
2. API 端点是否可访问
3. 请求头和数据格式是否正确（特别是 JSON 格式的头部信息）
4. 查看具体失败任务的日志以获取详细错误信息
5. 确保 secrets 名称完全匹配（区分大小写）