# 微信登录集成笔记

## Worker 集成步骤

### 1. 在 Worker 中初始化自动刷新服务

需要在 `core/src/core/worker.ts` 的 startBot 函数中添加:

```typescript
// 引入自动刷新服务
const { createAutoCodeRefreshService } = require('../runtime/auto-code-refresh');

// 在 startBot 函数中
async function startBot(config) {
  // ... 现有代码 ...
  
  // 初始化自动刷新服务
  if (!autoCodeRefreshService) {
    autoCodeRefreshService = createAutoCodeRefreshService({
      store: globalStore,
      getAccounts: () => getAccountsData(),
      addOrUpdateAccount: (account) => updateAccountData(account),
      resolveWorkerControls: () => ({
        restartWorker: (newAccount) => {
          // 重启 Worker 并应用新的 Code
          stopBot().then(() => {
            config.code = newAccount.code;
            startBot(config);
          });
        }
      }),
      log: (msg) => console.log('[AutoCodeRefresh]', msg),
      addAccountLog: (type, msg, accountId, accountName, data) => {
        // 记录账号日志
      }
    });
  }
  
  // 启动自动刷新
  if (autoCodeRefreshService) {
    autoCodeRefreshService.enableAutoCodeRefresh(config.accountId);
  }
}
```

### 2. 在 Worker 停止时清理自动刷新

需要在 `stopBot` 函数中添加:

```typescript
async function stopBot() {
  // ... 现有代码 ...
  
  // 停止自动刷新
  if (autoCodeRefreshService && config?.accountId) {
    autoCodeRefreshService.disableAutoCodeRefresh(config.accountId);
  }
  
  // ... 其他清理代码 ...
}
```

### 3. 登录路由集成

微信登录路由已在 `core/src/controllers/admin/wx-login-routes.ts` 中定义,
只需确保以下 API 端点可用:

- POST /api/wx-login/tasks - 创建登录任务
- GET /api/wx-login/tasks/:taskId/qr - 获取二维码
- GET /api/wx-login/tasks/:taskId/status - 轮询状态
- POST /api/wx-login/tasks/:taskId/confirm - 确认授权
- POST /api/wx-login/tasks/:taskId/code - 获取 Code
- DELETE /api/wx-login/tasks/:taskId - 删除任务

### 4. 配置更新

需要在 `core/data/global-config.json` 中添加微信登录配置:

```json
{
  "wxLoginConfig": {
    "oauthAppId": "wxd44977328b36e647",
    "targetAppId": "wx5306c5978fdb76e4",
    "sessionTtlMs": 300000,
    "qrConnectUrl": "https://open.weixin.qq.com/connect/qrconnect",
    "qrPollUrl": "https://long.open.weixin.qq.com/connect/l/qrconnect",
    "callbackUrl": "https://yybadaccess.3g.qq.com/pc_yyb/pcyyb_oauth",
    "loginBufferUrl": "https://yybadaccess.3g.qq.com/pc_yyb_auth/pcyyb_get_wx_login_buffer_auth"
  },
  "autoCodeRefreshConfig": {
    "enabled": true,
    "intervalMinutes": 60,
    "refreshThresholdMs": 1800000,
    "maxDailyRecoveries": 8,
    "maxConsecutiveFailures": 3
  }
}
```

## 测试验证

### 单元测试
```bash
npm test -- core/test/wx-login*.test.js
npm test -- core/test/auto-code-refresh.test.js
```

### 集成测试
```bash
npm test -- core/test/integration/
```

### 手动功能测试
1. 启动开发服务器: `npm run dev`
2. 打开管理面板: http://localhost:3007
3. 测试微信登录流程
4. 验证 Code 自动刷新

## 关键文件

- `core/src/services/wx-login-adapter.js` - 登录适配层 API
- `core/src/services/wx-login/service.js` - OAuth 流程实现
- `core/src/services/wx-login/native-protocol.js` - MMTLS 协议
- `core/src/runtime/auto-code-refresh.js` - 自动刷新服务
- `core/src/controllers/admin/wx-login-routes.ts` - 登录路由

## 故障排查

### 编译错误
- 检查类型定义是否正确导入
- 运行 `npm run build` 验证 TypeScript 编译

### 运行时错误
- 检查日志输出
- 验证依赖是否正确安装
- 检查配置参数是否正确

### 网络问题
- 验证网络连接
- 检查微信 OAuth URL 是否可访问
- 增加超时时间

