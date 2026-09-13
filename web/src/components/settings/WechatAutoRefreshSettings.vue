<script setup lang="ts">
import { onMounted, ref, computed } from 'vue'
import { NCard, NButton, NSpace, NSpin, NAlert, NTag, NSwitch } from 'naive-ui/es/index'
import api from '@/api'
import BaseInput from '@/components/ui/BaseInput.vue'
import BaseSwitch from '@/components/ui/BaseSwitch.vue'

interface WechatRefreshStatus {
  accountId: string
  accountName: string
  platform: string
  enabled: boolean
  intervalMinutes: number
  lastRefreshTime?: string
  lastRefreshStatus?: 'success' | 'failed' | 'pending'
  lastRefreshError?: string
  nextRefreshTime?: string
}

const loading = ref(false)
const saving = ref(false)
const refreshStatuses = ref<WechatRefreshStatus[]>([])
const configForm = ref({
  enabled: false,
  intervalMinutes: 60,
})

const wechatAccounts = computed(() => 
  refreshStatuses.value.filter(acc => acc.platform === 'wx')
)

const hasWechatAccounts = computed(() => wechatAccounts.value.length > 0)

onMounted(async () => {
  await loadRefreshStatus()
})

async function loadRefreshStatus() {
  loading.value = true
  try {
    const response = await api.get('/api/wechat-auto-refresh-status')
    if (response.data && response.data.ok) {
      refreshStatuses.value = response.data.data || []
    }
  } catch (error) {
    console.error('Failed to load wechat auto refresh status:', error)
  } finally {
    loading.value = false
  }
}

async function updateAccountRefreshConfig(accountId: string, config: any) {
  saving.value = true
  try {
    const response = await api.post(`/api/wechat-auto-refresh/${accountId}`, config)
    if (response.data && response.data.ok) {
      await loadRefreshStatus()
    }
  } catch (error) {
    console.error('Failed to update wechat auto refresh config:', error)
  } finally {
    saving.value = false
  }
}

async function triggerRefreshNow(accountId: string) {
  saving.value = true
  try {
    const response = await api.post(`/api/wechat-auto-refresh/${accountId}/refresh-now`)
    if (response.data && response.data.ok) {
      await loadRefreshStatus()
    }
  } catch (error) {
    console.error('Failed to trigger refresh:', error)
  } finally {
    saving.value = false
  }
}

function getRefreshStatusColor(status?: string) {
  switch (status) {
    case 'success':
      return 'success'
    case 'failed':
      return 'error'
    case 'pending':
      return 'warning'
    default:
      return 'default'
  }
}

function getRefreshStatusLabel(status?: string) {
  switch (status) {
    case 'success':
      return '成功'
    case 'failed':
      return '失败'
    case 'pending':
      return '进行中'
    default:
      return '未知'
  }
}
</script>

<template>
  <div class="space-y-4">
    <div v-if="!hasWechatAccounts" class="farm-card rounded-2xl p-4">
      <NAlert type="info" class="mb-4">
        暂无微信账号，此功能仅适用于通过微信扫码登录的账号。
      </NAlert>
    </div>

    <NSpin :show="loading">
      <div v-for="account in wechatAccounts" :key="account.accountId" class="farm-card rounded-2xl p-4">
        <div class="mb-4 flex items-start justify-between gap-3">
          <div class="flex items-start gap-3 flex-1">
            <div class="h-9 w-9 flex shrink-0 items-center justify-center rounded-lg bg-blue-50 text-blue-600 dark:bg-blue-900/25 dark:text-blue-400">
              <span class="i-carbon-connect text-xl" />
            </div>
            <div class="flex-1">
              <h4 class="text-base text-gray-900 font-bold dark:text-gray-100">
                {{ account.accountName }}
              </h4>
              <p class="mt-0.5 text-xs text-gray-500 dark:text-gray-400">
                微信自动刷新 Code
              </p>
            </div>
          </div>
          <NTag
            :type="getRefreshStatusColor(account.lastRefreshStatus)"
            size="small"
            round
          >
            {{ getRefreshStatusLabel(account.lastRefreshStatus) }}
          </NTag>
        </div>

        <!-- 启用开关 -->
        <div class="mb-4">
          <BaseSwitch 
            v-model="account.enabled"
            label="启用自动刷新"
            @update:model-value="() => updateAccountRefreshConfig(account.accountId, { enabled: account.enabled })"
          />
        </div>

        <!-- 刷新间隔设置 -->
        <div v-if="account.enabled" class="mb-4">
          <BaseInput 
            v-model.number="account.intervalMinutes"
            label="刷新间隔（分钟）"
            type="number"
            min="1"
            max="1440"
            @blur="() => updateAccountRefreshConfig(account.accountId, { intervalMinutes: account.intervalMinutes })"
          />
        </div>

        <!-- 状态信息 -->
        <div v-if="account.lastRefreshTime" class="mb-4 p-3 rounded-lg bg-gray-50 dark:bg-gray-900/30 space-y-2">
          <div class="flex justify-between text-sm">
            <span class="text-gray-600 dark:text-gray-400">最后刷新：</span>
            <span class="text-gray-900 dark:text-gray-100">{{ account.lastRefreshTime }}</span>
          </div>
          <div v-if="account.nextRefreshTime" class="flex justify-between text-sm">
            <span class="text-gray-600 dark:text-gray-400">下次刷新：</span>
            <span class="text-gray-900 dark:text-gray-100">{{ account.nextRefreshTime }}</span>
          </div>
          <div v-if="account.lastRefreshError" class="flex justify-between text-sm">
            <span class="text-red-600 dark:text-red-400">错误：</span>
            <span class="text-red-900 dark:text-red-100 text-right">{{ account.lastRefreshError }}</span>
          </div>
        </div>

        <!-- 操作按钮 -->
        <div class="flex gap-2">
          <NButton 
            :loading="saving"
            :disabled="saving || !account.enabled"
            @click="() => triggerRefreshNow(account.accountId)"
          >
            立即刷新
          </NButton>
          <NButton 
            text
            type="primary"
            @click="() => loadRefreshStatus()"
          >
            刷新状态
          </NButton>
        </div>
      </div>
    </NSpin>
  </div>
</template>

<style scoped>
.farm-card {
  background: var(--card-bg);
  border: 1px solid var(--border-color);
  transition: all 0.3s ease;
}

.farm-card:hover {
  border-color: var(--primary-color);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
