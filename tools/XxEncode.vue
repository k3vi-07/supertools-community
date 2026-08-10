<template>
  <h-single-layout>
    <div class="xxencode">
      <div class="xxencode__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 xxencode 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="xxencode__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> xxencode</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">xxencode -> 文本</button>
      </div>
      <div class="xxencode__output selectable">{{ output }}</div>
      <p class="xxencode__hint">xxencode 是 uuencode 的变体，使用 + 和 - 之间的字符，跨平台兼容性更好。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const mode = ref('encode')

const CHARSET = '+-0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz'

function encode(str) {
  const bytes = new TextEncoder().encode(str)
  let result = ''
  for (let i = 0; i < bytes.length; i += 3) {
    const remaining = Math.min(3, bytes.length - i)
    const b = [bytes[i] || 0, bytes[i + 1] || 0, bytes[i + 2] || 0]
    result += CHARSET[remaining]
    result += CHARSET[(b[0] >> 2) & 0x3f]
    result += CHARSET[((b[0] << 4) | (b[1] >> 4)) & 0x3f]
    result += CHARSET[((b[1] << 2) | (b[2] >> 6)) & 0x3f]
    result += CHARSET[b[2] & 0x3f]
  }
  return result
}

function decode(str) {
  const clean = str.replace(new RegExp(`[^${CHARSET}]`, 'g'), '')
  const bytes = []
  for (let i = 0; i < clean.length; i += 4) {
    const len = CHARSET.indexOf(clean[i])
    if (len <= 0) break
    const c = [clean[i+1], clean[i+2], clean[i+3], clean[i+4]].map(ch => {
      const idx = CHARSET.indexOf(ch)
      return idx < 0 ? 0 : idx
    })
    if (len >= 1) bytes.push((c[0] << 2) | (c[1] >> 4))
    if (len >= 2) bytes.push(((c[1] << 4) | (c[2] >> 2)) & 0xff)
    if (len >= 3) bytes.push(((c[2] << 6) | c[3]) & 0xff)
  }
  return new TextDecoder().decode(new Uint8Array(bytes))
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value) : decode(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.xxencode{display:flex;flex-direction:column;gap:12px}
.xxencode__field{display:flex;flex-direction:column;gap:4px}
.xxencode__field label{font-size:12px;color:var(--text-tertiary)}
.xxencode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.xxencode__actions{display:flex;gap:8px}
.xxencode__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.xxencode__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.xxencode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.xxencode__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
