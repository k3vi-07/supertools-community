<template>
  <h-single-layout>
    <div class="ip-loc">
      <div class="ip-loc__input">
        <input v-model="ip" placeholder="输入 IP 地址 (留空查询本机)" @keyup.enter="lookup" />
        <button @click="lookup">查询</button>
      </div>
      <div v-if="loading" class="ip-loc__loading">查询中...</div>
      <div v-if="data" class="ip-loc__result">
        <div class="ip-loc__row"><span>IP</span><code>{{ data.ip || ip }}</code></div>
        <div class="ip-loc__row"><span>国家/地区</span><strong>{{ data.country }} {{ data.countryCode }}</strong></div>
        <div class="ip-loc__row"><span>省/州</span><strong>{{ data.regionName }}</strong></div>
        <div class="ip-loc__row"><span>城市</span><strong>{{ data.city }}</strong></div>
        <div class="ip-loc__row"><span>运营商</span><strong>{{ data.org || data.isp }}</strong></div>
        <div class="ip-loc__row"><span>时区</span><code>{{ data.timezone }}</code></div>
        <div class="ip-loc__row"><span>经纬度</span><code>{{ data.lat }}, {{ data.lon }}</code></div>
        <div class="ip-loc__map" v-if="data.lat && data.lon">
          <iframe :src="'https://www.openstreetmap.org/export/embed.html?bbox=' + (data.lon-0.1) + ',' + (data.lat-0.1) + ',' + (data.lon+0.1) + ',' + (data.lat+0.1) + '&layer=mapicon&marker=' + data.lat + ',' + data.lon" width="100%" height="250" frameborder="0" style="border:0; border-radius:8px;"></iframe>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'
const ip = ref('')
const data = ref<Record<string, unknown> | null>(null)
const loading = ref(false)

async function lookup(): Promise<void> {
  loading.value = true
  data.value = null
  try {
    const url = ip.value.trim() ? `http://ip-api.com/json/${ip.value}?lang=zh-CN` : 'http://ip-api.com/json/?lang=zh-CN'
    const src = await window.supertools.fetchRemote(url)
    data.value = JSON.parse(src)
  } catch (err) {
    window.$he3?.message.error('查询失败: ' + (err as Error).message)
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.ip-loc { display: flex; flex-direction: column; gap: 16px; }
.ip-loc__input { display: flex; gap: 8px; }
.ip-loc__input input { flex: 1; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.ip-loc__input button { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.ip-loc__loading { text-align: center; padding: 20px; color: var(--text-tertiary); }
.ip-loc__result { display: flex; flex-direction: column; gap: 6px; }
.ip-loc__row { display: flex; justify-content: space-between; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; }
.ip-loc__row span { color: var(--text-tertiary); }
.ip-loc__row code { font-family: monospace; color: var(--color-primary); }
.ip-loc__map { margin-top: 8px; }
</style>
