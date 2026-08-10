<template>
  <h-single-layout>
    <div class="base91">
      <div class="base91__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Base91 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="base91__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> Base91</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">Base91 -> 文本</button>
      </div>
      <div class="base91__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const mode = ref('encode')

const CHARSET = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!#$%&()*+,./:;<=>?@[]^_`{|}~"'

function encodeBase91(str) {
  const bytes = new TextEncoder().encode(str)
  let result = ''
  let b = 0, n = 0, v = 0
  for (let i = 0; i < bytes.length; i++) {
    b |= bytes[i] << n
    n += 8
    if (n > 13) {
      v = b & 8191
      if (v > 88) { b >>= 13; n -= 13 } else { v = b & 16383; b >>= 14; n -= 14 }
      result += CHARSET[v % 91] + CHARSET[Math.floor(v / 91)]
    }
  }
  if (n) {
    result += CHARSET[b % 91]
    if (n > 7 || b > 90) result += CHARSET[Math.floor(b / 91)]
  }
  return result
}

function decodeBase91(str) {
  const clean = str.replace(/\s/g, '')
  const bytes = []
  let b = 0, n = 0, v = -1
  for (let i = 0; i < clean.length; i++) {
    const c = CHARSET.indexOf(clean[i])
    if (c < 0) continue
    if (v < 0) { v = c } else {
      v += c * 91
      b |= v << n
      n += (v & 8191) > 88 ? 13 : 14
      while (n > 7) { bytes.push(b & 255); b >>= 8; n -= 8 }
      v = -1
    }
  }
  if (v + 1) { b |= (v + 1) << n; bytes.push(b & 255) }
  return new TextDecoder().decode(new Uint8Array(bytes))
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encodeBase91(input.value) : decodeBase91(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.base91{display:flex;flex-direction:column;gap:12px}
.base91__field{display:flex;flex-direction:column;gap:4px}
.base91__field label{font-size:12px;color:var(--text-tertiary)}
.base91__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.base91__actions{display:flex;gap:8px}
.base91__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.base91__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.base91__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
