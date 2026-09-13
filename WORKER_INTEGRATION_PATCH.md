# Worker 集成补丁

## 步骤 1: 在 Worker 中添加自动刷新服务初始化

需要在 `core/src/core/worker.ts` 的顶部添加导入:

```typescript
const { createAutoCodeRefreshService } = require('../runtime/auto-code-refresh');
```

## 步骤 2: 在 Worker 全局作用域添加变量

```typescript
let autoCodeRefreshService: any = null;
let workerConfig: any = null;
```

## 步骤 3: 在 startBot 函数中初始化自动刷新

在 `startBot` 函数的现有启动代码之后添加:

```typescript
async function startBot(config) {
  // ... 现有代码 ...
  
  // 保存配置用于停止时使用
  workerConfig = config;
  
  // 初始化自动刷新服务（如果尚未初始化）
  if (!autoCodeRefreshService && config.accountId) {
    autoCodeRefreshService = createAutoCodeRefreshService({
      store: {
        getAutoCodeRefresh: (accountId) => {
          return {
            enabled: true,
            intervalMinutes: 60
          };
        }
      },
      getAccounts: () => {
        return { accounts: [config] };
      },
      addOrUpdateAccount: (newAccount) => {
        // 更新账号数据
        Object.assign(config, newAccount);
      },
      resolveWorkerControls: () => ({
        restartWorker: (newAccount) => {
          // 重启 Worker 以应用新的 Code
          console.log('[AutoCodeRefresh] Restarting worker with new code');
          stopBot().then(() => {
            config.code = newAccount.code;
            startBot(config);
          }).catch(err => {
            console.error('[AutoCodeRefresh] Error restarting worker:', err);
          });
        }
      }),
      log: (msg) => {
        console.log('[AutoCodeRefresh]', msg);
      },
      addAccountLog: (type, msg, accountId, accountName, data) => {
        console.log(`[${accountName}] ${msg}`);
      }
    });
    
    // 启动自动刷新
    if (autoCodeRefreshService && config.accountId) {
      autoCodeRefreshService.enableAutoCodeRefresh(config.accountId);
      console.log('[AutoCodeRefresh] Service enabled for account:', config.accountId);
    }
  }
}
```

## 步骤 4: 在 stopBot 函数中清理自动刷新

在 `stopBot` 函数的现有清理代码之前添加:

```typescript
async function stopBot() {
  // 停止自动刷新服务
  if (autoCodeRefreshService && workerConfig?.accountId) {
    autoCodeRefreshService.disableAutoCodeRefresh(workerConfig.accountId);
    console.log('[AutoCodeRefresh] Service disabled for account:', workerConfig.accountId);
  }
  
  // ... 现有停止代码 ...
}
```

## 步骤 5: 添加 IPC 消息处理

确保主进程与 Worker 之间的 IPC 通信支持 Code 更新:

```typescript
if (msg.type === 'code-update') {
  // 应用新的 Code
  if (config && msg.code) {
    config.code = msg.code;
    console.log('[IPC] Code updated from main process');
  }
}
```

## 验证步骤

完成集成后，执行以下验证:

```bash
# 编译检查
npm run build

# 查看是否有错误
npm run build 2>&1 | grep -i error
```

