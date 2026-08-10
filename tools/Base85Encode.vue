<template>
  <h-single-layout>
    <div class="base85">
      <div class="base85__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Base85 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="base85__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 → Base85</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">Base85 → 文本</button>
      </div>
      <div class="base85__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const mode = ref('encode')

const CHARSET = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz!#$%&()*+-;<=>?@^_`{|}~'

function encodeBase85(str) {
  const bytes = new TextEncoder().encode(str)
  let result = ''
  for (let i = 0; i < bytes.length; i += 4) {
    let n = 0
    let remaining = Math.min(4, bytes.length - i)
    for (let j = 0; j < 4; j++) {
      n = n * 256 + (j < remaining ? bytes[i + j] : 0)
    }
    const group = []
    for (let j = 0; j < 5; j++) {
      group.unshift(n % 85)
      n = Math.floor(n / 85)
    }
    let chunk = group.map(c => CHARSET[c]).join('')
    if (remaining < 4) chunk = chunk.slice(0, remaining + 1)
    result += chunk
  }
  return result
}

function decodeBase85(str) {
  const clean = str.replace(/\s/g, '')
  const bytes = []
  for (let i = 0; i < clean.length; i += 5) {
    let n = 0
    let remaining = Math.min(5, clean.length - i)
    for (let j = 0; j < 5; j++) {
      const ch = j < remaining ? clean[i + j] : 'u'
      const idx = CHARSET.indexOf(ch)
      if (idx < 0) throw new Error(`非法字符: ${ch}`)
      n = n * 85 + idx
    }
    const group = []
    for (let j = 0; j < 4; j++) {
      group.unshift(n % 256)
      n = Math.floor(n / 256)
    }
    const outLen = remaining < 5 ? remaining - 1 : 4
    for (let j = 0; j < outLen; j++) bytes.push(group[j])
  }
  return new TextDecoder().decode(new Uint8Array(bytes))
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encodeBase85(input.value) : decodeBase85(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.base85{display:flex;flex-direction:column;gap:12px}
.base85__field{display:flex;flex-direction:column;gap:4px}
.base85__field label{font-size:12px;color:var(--text-tertiary)}
.base85__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.base85__actions{display:flex;gap:8px}
.base85__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.base85__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.base85__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
