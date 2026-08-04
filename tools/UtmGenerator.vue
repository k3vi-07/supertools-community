<template>
  <h-single-layout>
    <div class="utm">
      <div class="utm__grid">
        <div class="utm__field"><label>URL *</label><input v-model="url" placeholder="https://example.com" /></div>
        <div class="utm__field"><label>utm_source *</label><input v-model="utm.source" placeholder="google" /></div>
        <div class="utm__field"><label>utm_medium</label><input v-model="utm.medium" placeholder="cpc" /></div>
        <div class="utm__field"><label>utm_campaign</label><input v-model="utm.campaign" placeholder="spring_sale" /></div>
        <div class="utm__field"><label>utm_term</label><input v-model="utm.term" placeholder="running_shoes" /></div>
        <div class="utm__field"><label>utm_content</label><input v-model="utm.content" placeholder="logolink" /></div>
      </div>
      <div class="utm__output">
        <div class="utm__header"><span>生成的 UTM URL</span><button v-if="result" @click="copy">复制</button></div>
        <pre class="utm__url selectable">{{ result || '请输入 URL 和至少一个 UTM 参数' }}</pre>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from 'vue'
const url = ref('https://supertools.app')
const utm = reactive({ source: 'google', medium: 'cpc', campaign: 'spring_sale', term: '', content: '' })

const result = computed(() => {
  if (!url.value || !utm.source) return ''
  const params = new URLSearchParams()
  params.set('utm_source', utm.source)
  if (utm.medium) params.set('utm_medium', utm.medium)
  if (utm.campaign) params.set('utm_campaign', utm.campaign)
  if (utm.term) params.set('utm_term', utm.term)
  if (utm.content) params.set('utm_content', utm.content)
  const sep = url.value.includes('?') ? '&' : '?'
  return url.value + sep + params.toString()
})

function copy(): void { window.$he3?.copyText(result.value); window.$he3?.message.success('已复制') }
</script>

<style scoped>
.utm { display: flex; flex-direction: column; gap: 16px; }
.utm__grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.utm__field { display: flex; flex-direction: column; gap: 4px; }
.utm__field label { font-size: 12px; color: var(--text-secondary); }
.utm__field input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; outline: none; }
.utm__field input:focus { border-color: var(--color-primary); }
.utm__output { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.utm__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.utm__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.utm__url { padding: 12px; font-family: monospace; font-size: 12px; color: var(--color-primary); background: var(--bg-code); word-break: break-all; white-space: pre-wrap; }
</style>
