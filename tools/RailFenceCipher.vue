<template>
  <h-single-layout>
    <div class="railfence">
      <div class="railfence__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入明文' : '输入密文'" spellcheck="false"></textarea>
      </div>
      <div class="railfence__controls">
        <label>栏数: <input type="number" v-model.number="rails" min="2" max="20" /></label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">加密</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解密</button>
      </div>
      <div class="railfence__output selectable">{{ output }}</div>
      <p class="railfence__hint">栅栏密码将明文按 Z 字形排列在指定栏数上，再逐行读取。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLO WORLD')
const mode = ref('encode')
const rails = ref(3)

function encode(text, n) {
  if (n < 2) return text
  const fence = Array.from({ length: n }, () => [])
  let rail = 0, dir = 1
  for (const ch of text) {
    fence[rail].push(ch)
    rail += dir
    if (rail === n - 1 || rail === 0) dir = -dir
  }
  return fence.flat().join('')
}

function decode(text, n) {
  if (n < 2) return text
  const pattern = Array.from({ length: n }, () => [])
  let rail = 0, dir = 1
  for (let i = 0; i < text.length; i++) {
    pattern[rail].push(i)
    rail += dir
    if (rail === n - 1 || rail === 0) dir = -dir
  }
  const indices = pattern.flat()
  const result = new Array(text.length)
  indices.forEach((idx, i) => { result[idx] = text[i] })
  return result.join('')
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value, rails.value) : decode(input.value, rails.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.railfence{display:flex;flex-direction:column;gap:12px}
.railfence__field{display:flex;flex-direction:column;gap:4px}
.railfence__field label{font-size:12px;color:var(--text-tertiary)}
.railfence__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.railfence__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary)}
.railfence__controls input{width:60px;padding:4px 8px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);color:var(--text-primary)}
.railfence__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.railfence__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.railfence__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.railfence__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
