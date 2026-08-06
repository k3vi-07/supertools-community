<template>
  <h-single-layout>
    <div class="http-tester">
      <div class="http-tester__bar">
        <select v-model="method" class="http-tester__method">
          <option>GET</option><option>POST</option><option>PUT</option>
          <option>DELETE</option><option>PATCH</option><option>HEAD</option>
        </select>
        <input v-model="url" class="http-tester__url" placeholder="https://api.example.com/data" />
        <button class="http-tester__send" :disabled="loading" @click="send">{{ loading ? '...' : '发送' }}</button>
      </div>
      <div class="http-tester__section">
        <label>Headers (JSON)</label>
        <textarea v-model="headers" class="http-tester__textarea" rows="3" placeholder='{"Authorization": "Bearer xxx"}' spellcheck="false"></textarea>
      </div>
      <div v-if="method !== 'GET'" class="http-tester__section">
        <label>Body</label>
        <textarea v-model="body" class="http-tester__textarea" rows="4" placeholder="请求体..." spellcheck="false"></textarea>
      </div>
      <div v-if="response" class="http-tester__result">
        <div class="http-tester__status" :class="statusClass">
          {{ response.status }} {{ response.statusText }} · {{ response.time }}ms
        </div>
        <pre class="http-tester__output selectable">{{ response.body }}</pre>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const method = ref('GET')
const url = ref('https://httpbin.org/get')
const headers = ref('')
const body = ref('')
const loading = ref(false)
const response = ref(null)

const statusClass = computed(() => {
  if (!response.value) return ''
  const s = response.value.status
  if (s >= 200 && s < 300) return 'ok'
  if (s >= 400) return 'err'
  return 'warn'
})

async function send() {
  if (!url.value) { window.$he3?.message.error('请输入 URL'); return }
  loading.value = true
  response.value = null
  const start = Date.now()
  try {
    const opts = { method: method.value }
    const parsedHeaders = headers.value ? JSON.parse(headers.value) : {}
    if (Object.keys(parsedHeaders).length > 0) opts.headers = parsedHeaders
    if (method.value !== 'GET' && body.value) {
      opts.headers = opts.headers || {}
      if (!opts.headers['Content-Type']) opts.headers['Content-Type'] = 'application/json'
      opts.body = body.value
    }
    const res = await fetch(url.value, opts)
    const text = await res.text()
    let formatted = text
    try { formatted = JSON.stringify(JSON.parse(text), null, 2) } catch { /* not JSON */ }
    response.value = {
      status: res.status,
      statusText: res.statusText,
      time: Date.now() - start,
      body: formatted
    }
  } catch (err) {
    response.value = {
      status: 0,
      statusText: 'Network Error',
      time: Date.now() - start,
      body: '❌ ' + err.message
    }
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.http-tester { display: flex; flex-direction: column; gap: 12px; }
.http-tester__bar { display: flex; gap: 8px; }
.http-tester__method { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-weight: 600; }
.http-tester__url { flex: 1; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); outline: none; }
.http-tester__send { padding: 8px 20px; border: none; border-radius: 8px; background: var(--color-primary); color: white; cursor: pointer; font-weight: 600; }
.http-tester__send:disabled { opacity: 0.5; }
.http-tester__section label { display: block; font-size: 12px; color: var(--text-tertiary); margin-bottom: 4px; }
.http-tester__textarea { width: 100%; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.http-tester__status { font-size: 13px; font-weight: 600; padding: 6px 12px; border-radius: 6px; }
.http-tester__status.ok { background: rgba(34,197,94,0.15); color: #22c55e; }
.http-tester__status.err { background: rgba(239,68,68,0.15); color: #ef4444; }
.http-tester__status.warn { background: rgba(245,158,11,0.15); color: #f59e0b; }
.http-tester__output { padding: 12px; border-radius: 8px; background: var(--bg-base); font-size: 12px; max-height: 300px; overflow: auto; white-space: pre-wrap; word-break: break-all; }
</style>
