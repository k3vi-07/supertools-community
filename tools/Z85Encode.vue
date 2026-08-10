<template>
  <h-single-layout>
    <div class="z85">
      <div class="z85__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Z85 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="z85__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> Z85</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">Z85 -> 文本</button>
      </div>
      <div class="z85__output selectable">{{ output }}</div>
      <p class="z85__hint">Z85 是 ZeroMQ 的 Base85 变体，要求输入长度为 4 的倍数。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World!!')
const mode = ref('encode')

const CHARSET = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ.-:+=^!/*?&<>()[]{}@%$#'

function encode(str) {
  const bytes = new TextEncoder().encode(str)
  if (bytes.length % 4 !== 0) return '❌ 编码要求字节长度为 4 的倍数'
  let result = ''
  for (let i = 0; i < bytes.length; i += 4) {
    let n = 0
    for (let j = 0; j < 4; j++) n = n * 256 + bytes[i + j]
    const chars = []
    for (let j = 0; j < 5; j++) { chars.unshift(CHARSET[n % 85]); n = Math.floor(n / 85) }
    result += chars.join('')
  }
  return result
}

function decode(str) {
  const clean = str.replace(/\s/g, '')
  if (clean.length % 5 !== 0) return '❌ 解码要求字符串长度为 5 的倍数'
  const bytes = []
  for (let i = 0; i < clean.length; i += 5) {
    let n = 0
    for (let j = 0; j < 5; j++) {
      const idx = CHARSET.indexOf(clean[i + j])
      if (idx < 0) throw new Error(`非法字符: ${clean[i+j]}`)
      n = n * 85 + idx
    }
    for (let j = 3; j >= 0; j--) { bytes.push((n >> (j * 8)) & 0xff); n = Math.floor(n / 256) }
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
.z85{display:flex;flex-direction:column;gap:12px}
.z85__field{display:flex;flex-direction:column;gap:4px}
.z85__field label{font-size:12px;color:var(--text-tertiary)}
.z85__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.z85__actions{display:flex;gap:8px}
.z85__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.z85__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.z85__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.z85__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
