<template>
  <h-single-layout>
    <div class="xss-san">
      <div class="xss-san__controls">
        <label class="xss-san__chk"><input type="checkbox" v-model="stripScript" /> 移除 script 标签</label>
        <label class="xss-san__chk"><input type="checkbox" v-model="stripEvents" /> 移除事件属性</label>
        <label class="xss-san__chk"><input type="checkbox" v-model="stripTags" /> 移除危险标签</label>
        <label class="xss-san__chk"><input type="checkbox" v-model="encodeQuotes" /> 编码引号</label>
      </div>
      <div class="xss-san__cols">
        <div class="xss-san__col">
          <label>原始 HTML</label>
          <textarea v-model="input" class="xss-san__textarea" spellcheck="false"></textarea>
        </div>
        <div class="xss-san__col">
          <label>净化结果</label>
          <div class="xss-san__output selectable">{{ output }}</div>
        </div>
      </div>
      <div v-if="threats.length" class="xss-san__threats">
        <span class="xss-san__threats-label">⚠️ 检测到威胁:</span>
        <span v-for="t in threats" :key="t" class="xss-san__threat-tag">{{ t }}</span>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('<div onclick="alert(1)">Hello</div>\n<script>evil()</script>\n<img src=x onerror=alert(1)>')
const stripScript = ref(true)
const stripEvents = ref(true)
const stripTags = ref(true)
const encodeQuotes = ref(false)

const dangerousTags = ['script', 'iframe', 'object', 'embed', 'svg', 'math', 'base', 'meta', 'link']

const output = computed(() => {
  let s = input.value
  const threats = []

  if (stripScript.value) {
    if (/<script[\s\S]*?<\/script>/gi.test(s)) threats.push('script 标签')
    s = s.replace(/<script[\s\S]*?<\/script>/gi, '')
  }

  if (stripEvents.value) {
    if (/\son\w+\s*=/gi.test(s)) threats.push('事件属性')
    s = s.replace(/\son\w+\s*=\s*("[^"]*"|'[^']*'|[^\s>]+)/gi, '')
  }

  if (stripTags.value) {
    const tagRegex = new RegExp(`<(${dangerousTags.join('|')})[\\s\\S]*?<\\/\\1>`, 'gi')
    if (tagRegex.test(s)) threats.push('危险标签')
    s = s.replace(new RegExp(`<(${dangerousTags.join('|')})\\b[^>]*[\\s\\S]*?<\\/\\1>`, 'gi'), '')
    s = s.replace(new RegExp(`<(${dangerousTags.join('|')})\\b[^>]*\\/?>`, 'gi'), '')
    if (/javascript:/gi.test(s)) threats.push('javascript: 协议')
    s = s.replace(/javascript:/gi, '')
  }

  if (encodeQuotes.value) {
    s = s.replace(/"/g, '&quot;').replace(/'/g, '&#39;')
  }

  return s
})

const threats = computed(() => {
  const s = input.value
  const found = []
  if (stripScript.value && /<script[\s\S]*?<\/script>/gi.test(s)) found.push('script')
  if (stripEvents.value && /\son\w+\s*=/gi.test(s)) found.push('on* 事件')
  if (/javascript:/gi.test(s)) found.push('javascript:')
  if (/<iframe|<object|<embed/gi.test(s)) found.push('iframe/object/embed')
  return found
})
</script>

<style scoped>
.xss-san { display: flex; flex-direction: column; gap: 12px; }
.xss-san__controls { display: flex; gap: 16px; flex-wrap: wrap; }
.xss-san__chk { font-size: 13px; color: var(--text-secondary); display: flex; align-items: center; gap: 4px; cursor: pointer; }
.xss-san__cols { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.xss-san__col { display: flex; flex-direction: column; gap: 4px; }
.xss-san__col label { font-size: 12px; color: var(--text-tertiary); }
.xss-san__textarea { width: 100%; min-height: 150px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; resize: vertical; outline: none; }
.xss-san__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 150px; white-space: pre-wrap; word-break: break-all; }
.xss-san__threats { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; padding: 8px 12px; border-radius: 8px; background: rgba(239,68,68,0.1); }
.xss-san__threats-label { font-size: 13px; color: #ef4444; font-weight: 600; }
.xss-san__threat-tag { font-size: 11px; padding: 2px 8px; border-radius: 4px; background: rgba(239,68,68,0.2); color: #ef4444; }
</style>
