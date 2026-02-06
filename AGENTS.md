# AGENTS.md - Subscription Manager (Cloudflare Workers)

## 项目概述

基于 Cloudflare Workers 的轻量级订阅管理系统，支持多渠道通知（TG、Webhook、企业微信、邮件等）、农历日期显示和财务管理功能。

- **主文件**: `index.js` (~350KB)
- **运行环境**: Cloudflare Workers
- **配置**: `wrangler.toml`
- **依赖管理**: `package.json`

## 构建与部署命令

### 本地开发
```bash
# 安装 Wrangler CLI
npm install -g wrangler

# 登录 Cloudflare
npx wrangler login

# 本地预览
npx wrangler dev

# 部署到生产环境
npx wrangler deploy

# 部署到 staging 环境
npx wrangler deploy --env staging
```

### 环境变量配置
通过 `wrangler.toml` 或 Cloudflare Dashboard 设置：
- `ENVIRONMENT`: 运行环境 (production/staging)
- `SUBSCRIPTIONS_KV`: KV 命名空间

### 定时任务
在 `wrangler.toml` 中配置 Cron 触发器：
```toml
[triggers]
crons = ["0 8 * * *"]  # 每天 UTC 8 点执行
```

## 代码风格规范

### 命名约定
- **变量/函数**: camelCase (如 `getCurrentTimeInTimezone`)
- **常量**: UPPER_SNAKE_CASE (如 `MS_PER_HOUR`, `MS_PER_DAY`)
- **对象/模块**: camelCase 或 PascalCase (如 `lunarCalendar`, `lunarBiz`)
- **DOM 元素 ID**: camelCase (如 `subscriptionModal`)

### 注释规范
- 使用中文注释和中文提示信息
- 函数头部添加功能说明注释
- 复杂逻辑添加行内注释

### 错误处理
- 使用 `try-catch` 包裹可能出错的代码
- 错误信息使用中文: `console.error(\`错误描述: ${error.message}\`)`
- 发生错误时返回合理的默认值或回退逻辑

### 代码格式
- 使用模板字符串 (backtick) 包裹 HTML 和 CSS
- 字符串优先使用单引号
- 缩进 2 空格 (已存在于代码中)
- 运算符两侧添加空格

### 变量声明
- 优先使用 `const`，可变变量使用 `let`
- 禁止使用 `var`

### 模块组织
- 工具函数放在文件顶部
- 常量定义紧跟其后
- 主逻辑放在模块定义之后
- CSS 样式作为字符串常量

## 关键文件结构

```
subscription-manager/
├── index.js           # 主应用文件 (所有路由、逻辑、UI)
├── wrangler.toml     # Cloudflare Workers 配置
├── package.json      # 项目元数据
└── README.md         # 项目文档
```

## 路由与端点

### 管理端
- `/admin` - 订阅列表页面
- `/admin/dashboard` - 仪表盘页面
- `/admin/config` - 系统配置页面
- `/api/login` - 登录
- `/api/logout` - 退出登录

### API 端点
- `/api/subscriptions` - 订阅 CRUD
- `/api/notify/{token}` - 触发通知
- `/api/config` - 配置管理
- `/api/stats` - 统计数据

## KV 命名空间

```toml
[[kv_namespaces]]
binding = "SUBSCRIPTIONS_KV"
id = "7c10ca7e173c4dc39f7f4098220ef219"
preview_id = "your-preview-kv-namespace-id"
```

## 测试说明

**注意**: 当前项目没有自动化测试框架。如需添加测试：

```bash
# 安装测试框架
npm install --save-dev vitest

# 运行测试
npx vitest

# 运行单文件测试
npx vitest run src/xxx.test.js
```

## 第三方服务集成

### 通知渠道
- **Telegram Bot**: 通过 BotFather 获取 Token
- **NotifyX**: API Key 认证
- **Webhook**: 支持自定义请求格式和模板
- **企业微信**: 群机器人 Webhook
- **Resend**: 邮件发送服务
- **Bark**: iOS 推送服务

### 农历支持
内置农历转换工具 (`lunarCalendar` 对象)：
- 支持 1900-2100 年
- 提供天干地支、月份、日期转换
- `lunarBiz` 模块提供农历业务逻辑

## 常用操作

### 添加新订阅类型
在 `index.js` 中搜索订阅类型下拉列表的选项，添加新类型。

### 修改默认账号
在 `index.js` 中查找默认用户名和密码 (admin/password 或 admin123)。

### 配置新的通知渠道
1. 在配置页面添加 API Key
2. 修改通知发送逻辑 (搜索 `sendNotification` 相关函数)

### 时区配置
- 系统默认使用 `Asia/Shanghai` (中国标准时间)
- 可通过 `X-Timezone` 请求头或 URL 参数覆盖
- 支持 18+ 个常用时区

## 注意事项

1. **Cloudflare Workers 限制**: 无状态，所有状态存储在 KV 中
2. **兼容性**: 使用 `nodejs_compat` 兼容标志
3. **安全性**: API 令牌通过 Bearer Token 或 URL 参数传递
4. **性能**: 尽量减少外部 API 调用，缓存常用数据
5. **备份**: 定期导出 KV 数据作为备份
