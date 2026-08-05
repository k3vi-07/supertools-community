<template>
  <h-single-layout>
    <div class="ping">
      <div class="ping__input"><input v-model="host" placeholder="输入域名或 IP..." @keyup.enter="ping" /><button @click="ping">Ping</button></div>
      <div v-if="loading" class="ping__loading">Ping 中...</div>
      <div v-if="results.length" class="ping__results">
        <div class="ping__summary">平均延迟: <strong>{{ avgLatency }}ms</strong> | 最小: {{ minLatency }}ms | 最大: {{ maxLatency }}ms</div>
        <div v-for="(r, i) in results" :key="i" class="ping__row" :class="{fail: !r.ok}">
          <span class="ping__seq">#{{ i+1 }}</span>
          <span class="ping__time" v-if="r.ok">{{ r.latency }}ms</span>
          <span class="ping__time" v-else>超时</span>
          <div class="ping__bar" v-if="r.ok"><div class="ping__bar-fill" :style="{width: Math.min(100, r.latency/10)+'%'}"></div></div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const host = ref('baidu.com')
const results = ref([])
const loading = ref(false)

const latencies = computed(() => results.value.filter((r) => r.ok).map((r) => r.latency))
const avgLatency = computed(() => latencies.value.length ? Math.round(latencies.value.reduce((a, b) => a + b, 0) / latencies.value.length) : 0)
const minLatency = computed(() => latencies.value.length ? Math.min(...latencies.value) : 0)
const maxLatency = computed(() => latencies.value.length ? Math.max(...latencies.value) : 0)

async function ping() {
  loading.value = true
  results.value = []
  // 使用 fetch 计时模拟 ping（受 CORS 限制，仅为近似值）
  for (let i = 0; i < 5; i++) {
    const start = performance.now()
    try {
      const controller = new AbortController()
      const timeout = setTimeout(() => controller.abort(), 3000)
      await fetch(`https://${host.value}/favicon.ico?t=${Date.now()}`, { mode: 'no-cors', signal: controller.signal, cache: 'no-store' })
      clearTimeout(timeout)
      const latency = Math.round(performance.now() - start)
      results.value.push({ ok: true, latency })
    } catch {
      results.value.push({ ok: false, latency: 0 })
    }
    await new Promise((r) => setTimeout(r, 500))
  }
  loading.value = false
}
</script>

<style scoped>
.ping { display: flex; flex-direction: column; gap: 16px; }
.ping__input { display: flex; gap: 8px; }
.ping__input input { flex: 1; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.ping__input button { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.ping__loading { text-align: center; color: var(--text-tertiary); }
.ping__summary { padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; color: var(--text-secondary); margin-bottom: 8px; }
.ping__summary strong { color: var(--color-primary); }
.ping__results { display: flex; flex-direction: column; gap: 4px; }
.ping__row { display: flex; align-items: center; gap: 12px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; }
.ping__row.fail { border-color: var(--color-error); }
.ping__seq { width: 40px; color: var(--text-tertiary); }
.ping__time { width: 80px; font-family: monospace; color: var(--color-primary); }
.ping__row.fail .ping__time { color: var(--color-error); }
.ping__bar { flex: 1; height: 6px; border-radius: 3px; background: var(--bg-active); overflow: hidden; }
.ping__bar-fill { height: 100%; background: var(--color-success); border-radius: 3px; }
</style>
