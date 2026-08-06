<template>
  <h-single-layout>
    <div class="str-esc">
      <div class="str-esc__input-section">
        <textarea v-model="input" class="str-esc__input" placeholder="输入文本..." spellcheck="false"></textarea>
      </div>
      <div class="str-esc__modes">
        <button v-for="m in modes" :key="m.id" class="str-esc__mode" :class="{active: mode === m.id}" @click="mode = m.id">{{ m.name }}</button>
      </div>
      <div class="str-esc__output-section">
        <div class="str-esc__output selectable">{{ output }}</div>
        <button class="str-esc__copy" @click="copy">复制</button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello "World"\n<svg/onload=alert(1)>\nLine\\Two\tTab')
const mode = ref('all')

const modes = [
  { id: 'all', name: '全部转义' },
  { id: 'html', name: 'HTML 实体' },
  { id: 'url', name: 'URL 编码' },
  { id: 'unicode', name: 'Unicode' },
  { id: 'js', name: 'JS 字符串' },
  { id: 'hex', name: 'Hex' },
]

const output = computed(() => {
  const s = input.value
  switch (mode.value) {
    case 'all':
      return s.split('').map(c => {
        const code = c.charCodeAt(0)
        if (code < 32 || code > 126) return '\\u' + code.toString(16).padStart(4, '0')
        return c
      }).join('')
    case 'html':
      return s.replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]))
    case 'url':
      return encodeURIComponent(s)
    case 'unicode':
      return s.split('').map(c => '\\u' + c.charCodeAt(0).toString(16).padStart(4, '0')).join('')
    case 'js':
      return JSON.stringify(s)
    case 'hex':
      return s.split('').map(c => '\\x' + c.charCodeAt(0).toString(16).padStart(2, '0')).join('')
    default:
      return s
  }
})

function copy() {
  window.$he3?.copyText(output.value)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.str-esc { display: flex; flex-direction: column; gap: 12px; }
.str-esc__input { width: 100%; min-height: 80px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; resize: vertical; outline: none; }
.str-esc__modes { display: flex; gap: 6px; flex-wrap: wrap; }
.str-esc__mode { padding: 6px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; font-size: 12px; }
.str-esc__mode.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.str-esc__output-section { position: relative; }
.str-esc__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 80px; white-space: pre-wrap; word-break: break-all; }
.str-esc__copy { position: absolute; top: 8px; right: 8px; padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; font-size: 12px; }
.str-esc__copy:hover { border-color: var(--color-primary); color: var(--color-primary); }
</style>
