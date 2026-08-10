<template>
  <h-single-layout>
    <div class="uuencode">
      <div class="uuencode__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 uuencode 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="uuencode__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> uuencode</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">uuencode -> 文本</button>
      </div>
      <div class="uuencode__output selectable">{{ output }}</div>
      <p class="uuencode__hint">uuencode 是 Unix 系统中用于将二进制文件编码为文本格式的经典算法。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const mode = ref('encode')

function encode(str) {
  const bytes = new TextEncoder().encode(str)
  let result = ''
  for (let i = 0; i < bytes.length; i += 3) {
    const remaining = Math.min(3, bytes.length - i)
    const b = [bytes[i] || 0, bytes[i + 1] || 0, bytes[i + 2] || 0]
    result += String.fromCharCode((remaining & 0x3f) + 0x20)
    if (remaining >= 1) result += String.fromCharCode((((b[0] >> 2) & 0x3f)) + 0x20)
    if (remaining >= 1) result += String.fromCharCode((((b[0] << 4) & 0x30) | ((b[1] >> 4) & 0x0f) | 0) + 0x20)
    if (remaining >= 2) result += String.fromCharCode((((b[1] << 2) & 0x3c) | ((b[2] >> 6) & 0x03) | 0) + 0x20)
    if (remaining >= 3) result += String.fromCharCode((b[2] & 0x3f) + 0x20)
  }
  return result
}

function decode(str) {
  const clean = str.replace(/[^ -_]/g, '').replace(/`/g, ' ')
  const bytes = []
  for (let i = 0; i < clean.length; i += 4) {
    const len = (clean.charCodeAt(i) - 0x20) & 0x3f
    if (len === 0) break
    const c = [
      (clean.charCodeAt(i + 1) || 0x20) - 0x20,
      (clean.charCodeAt(i + 2) || 0x20) - 0x20,
      (clean.charCodeAt(i + 3) || 0x20) - 0x20,
      (clean.charCodeAt(i + 4) || 0x20) - 0x20,
    ].map(v => v & 0x3f)
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
.uuencode{display:flex;flex-direction:column;gap:12px}
.uuencode__field{display:flex;flex-direction:column;gap:4px}
.uuencode__field label{font-size:12px;color:var(--text-tertiary)}
.uuencode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.uuencode__actions{display:flex;gap:8px}
.uuencode__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.uuencode__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.uuencode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.uuencode__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
