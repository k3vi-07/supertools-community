<template>
  <h-single-layout>
    <div class="http-status">
      <input v-model="search" placeholder="搜索状态码或名称... (如 404, Not Found, 重定向)" />

      <div class="http-status__groups">
        <div v-for="g in filteredGroups" :key="g.class" class="http-status__group">
          <div class="http-status__group-header" :style="{ borderColor: g.color }">
            <span class="http-status__class" :style="{ color: g.color }">{{ g.class }}xx</span>
            <span class="http-status__class-name">{{ g.name }}</span>
          </div>
          <div class="http-status__list">
            <div v-for="s in g.codes" :key="s.code" class="http-status__item" @click="copy(s)">
              <span class="http-status__code" :style="{ color: g.color }">{{ s.code }}</span>
              <span class="http-status__name">{{ s.name }}</span>
              <span class="http-status__desc">{{ s.desc }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const search = ref('')

const groups = [
  { class: 1, name: '信息响应', color: '#6b7280',
    codes: [
      { code: 100, name: 'Continue', desc: '继续发送请求体' },
      { code: 101, name: 'Switching Protocols', desc: '切换协议（如 WebSocket）' },
      { code: 102, name: 'Processing', desc: '服务器已收到请求，正在处理' }
    ]
  },
  { class: 2, name: '成功响应', color: '#22c55e',
    codes: [
      { code: 200, name: 'OK', desc: '请求成功' },
      { code: 201, name: 'Created', desc: '资源创建成功' },
      { code: 202, name: 'Accepted', desc: '请求已接受，异步处理' },
      { code: 204, name: 'No Content', desc: '成功但无返回内容' },
      { code: 206, name: 'Partial Content', desc: '部分内容（范围请求）' }
    ]
  },
  { class: 3, name: '重定向', color: '#3b82f6',
    codes: [
      { code: 301, name: 'Moved Permanently', desc: '永久重定向' },
      { code: 302, name: 'Found', desc: '临时重定向' },
      { code: 304, name: 'Not Modified', desc: '资源未修改，用缓存' },
      { code: 307, name: 'Temporary Redirect', desc: '临时重定向（保持方法）' },
      { code: 308, name: 'Permanent Redirect', desc: '永久重定向（保持方法）' }
    ]
  },
  { class: 4, name: '客户端错误', color: '#f59e0b',
    codes: [
      { code: 400, name: 'Bad Request', desc: '请求参数错误' },
      { code: 401, name: 'Unauthorized', desc: '未认证（需要登录）' },
      { code: 403, name: 'Forbidden', desc: '无权限访问' },
      { code: 404, name: 'Not Found', desc: '资源不存在' },
      { code: 405, name: 'Method Not Allowed', desc: '请求方法不允许' },
      { code: 408, name: 'Request Timeout', desc: '请求超时' },
      { code: 409, name: 'Conflict', desc: '请求冲突' },
      { code: 413, name: 'Payload Too Large', desc: '请求体过大' },
      { code: 415, name: 'Unsupported Media Type', desc: '不支持的媒体类型' },
      { code: 422, name: 'Unprocessable Entity', desc: '语义错误（验证失败）' },
      { code: 429, name: 'Too Many Requests', desc: '请求过多（限流）' }
    ]
  },
  { class: 5, name: '服务端错误', color: '#ef4444',
    codes: [
      { code: 500, name: 'Internal Server Error', desc: '服务器内部错误' },
      { code: 501, name: 'Not Implemented', desc: '服务器不支持此功能' },
      { code: 502, name: 'Bad Gateway', desc: '网关错误' },
      { code: 503, name: 'Service Unavailable', desc: '服务不可用（维护中）' },
      { code: 504, name: 'Gateway Timeout', desc: '网关超时' },
      { code: 511, name: 'Network Authentication Required', desc: '需要网络认证（如 WiFi）' }
    ]
  }
]

const filteredGroups = computed(() => {
  if (!search.value.trim()) return groups
  const q = search.value.toLowerCase()
  return groups
    .map(g => ({ ...g, codes: g.codes.filter(s =>
      String(s.code).includes(q) || s.name.toLowerCase().includes(q) || s.desc.includes(q)
    )}))
    .filter(g => g.codes.length > 0)
})

function copy(s) {
  navigator.clipboard?.writeText(String(s.code))
}
</script>

<style scoped>
.http-status { display: flex; flex-direction: column; gap: 12px; }
.http-status input { padding: 10px 14px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.http-status__groups { display: flex; flex-direction: column; gap: 16px; }
.http-status__group-header { display: flex; align-items: center; gap: 8px; margin-bottom: 8px; padding-left: 12px; border-left: 4px solid; }
.http-status__class { font-size: 18px; font-weight: 700; }
.http-status__class-name { font-size: 13px; color: var(--text-secondary); }
.http-status__list { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 8px; }
.http-status__item { display: flex; align-items: center; gap: 10px; padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.http-status__item:hover { border-color: var(--color-primary); transform: translateY(-1px); }
.http-status__code { font-size: 18px; font-weight: 700; font-family: monospace; min-width: 42px; }
.http-status__name { font-size: 13px; font-weight: 500; color: var(--text-primary); min-width: 120px; }
.http-status__desc { font-size: 12px; color: var(--text-tertiary); flex: 1; }
</style>
