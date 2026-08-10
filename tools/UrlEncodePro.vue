<template>
  <h-single-layout>
    <div class="url-decode-pro">
      <div class="url-decode-pro__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 URL 编码文本'" spellcheck="false"></textarea>
      </div>
      <div class="url-decode-pro__controls">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
        <label><input type="checkbox" v-model="encodeAll"> 编码所有字符</label>
        <label><input type="checkbox" v-model="usePlus"> 空格用 +</label>
      </div>
      <div class="url-decode-pro__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('https://example.com/搜索?q=hello world&lang=zh')
const mode = ref('encode')
const encodeAll = ref(false)
const usePlus = ref(false)

const SAFE = new Set('ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_.~')

function encode(str) {
  const bytes = new TextEncoder().encode(str)
  let result = ''
  for (const b of bytes) {
    if (!encodeAll.value && SAFE.has(String.fromCharCode(b))) {
      result += String.fromCharCode(b)
    } else if (usePlus.value && b === 0x20) {
      result += '+'
    } else {
      result += '%' + b.toString(16).toUpperCase().padStart(2, '0')
    }
  }
  return result
}

function decode(str) {
  if (usePlus.value) str = str.replace(/\+/g, '%20')
  try {
    return decodeURIComponent(str)
  } catch {
    // 处理不完整的百分号编码
    return str.replace(/%([0-9a-fA-F]{2})/g, (_, h) => String.fromCharCode(parseInt(h, 16)))
  }
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value) : decode(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.url-decode-pro{display:flex;flex-direction:column;gap:12px}
.url-decode-pro__field{display:flex;flex-direction:column;gap:4px}
.url-decode-pro__field label{font-size:12px;color:var(--text-tertiary)}
.url-decode-pro__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.url-decode-pro__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.url-decode-pro__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.url-decode-pro__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.url-decode-pro__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.url-decode-pro__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
</style>
