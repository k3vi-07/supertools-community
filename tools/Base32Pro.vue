<template>
  <h-single-layout>
    <div class="b32">
      <div class="b32__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Base32 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="b32__controls">
        <label><input type="checkbox" v-model="padding"> 填充 =</label>
        <label><input type="checkbox" v-model="hex"> 十六进制变体</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="b32__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')
const padding = ref(true)
const hex = ref(false)

const RFC = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567'
const HEX = '0123456789ABCDEFGHIJKLMNOPQRSTUV'

function encode(str) {
  const charset = hex.value ? HEX : RFC
  const bytes = new TextEncoder().encode(str)
  let result = ''
  for (let i = 0; i < bytes.length; i += 5) {
    const buf = [bytes[i]||0, bytes[i+1]||0, bytes[i+2]||0, bytes[i+3]||0, bytes[i+4]||0]
    const remaining = Math.min(5, bytes.length - i)
    const groups = [0,0,0,2,4]
    const outCount = [8,2,4,5,7][remaining]
    let bits = 0, n = 0
    for (const b of buf) { n = (n << 8) | b; bits += 8 }
    const chars = []
    for (let j = 0; j < 8; j++) {
      if (bits >= 5) { bits -= 5; chars.push(charset[(n >> bits) & 0x1f]) }
    }
    result += chars.slice(0, outCount).join('')
    if (padding.value) result += '='.repeat(8 - outCount)
  }
  return result
}

function decode(str) {
  const charset = hex.value ? HEX : RFC
  const clean = str.replace(/=/g, '').replace(/\s/g, '').toUpperCase()
  let bits = 0, n = 0
  const bytes = []
  for (const ch of clean) {
    const idx = charset.indexOf(ch)
    if (idx < 0) continue
    n = (n << 5) | idx
    bits += 5
    while (bits >= 8) { bits -= 8; bytes.push((n >> bits) & 0xff) }
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
.b32{display:flex;flex-direction:column;gap:12px}
.b32__field{display:flex;flex-direction:column;gap:4px}
.b32__field label{font-size:12px;color:var(--text-tertiary)}
.b32__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.b32__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.b32__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.b32__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.b32__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.b32__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
