<template>
  <h-single-layout>
    <div class="bifid">
      <div class="bifid__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入明文（仅字母）' : '输入密文'" spellcheck="false"></textarea>
      </div>
      <div class="bifid__controls">
        <label>密钥: <input type="text" v-model="key" placeholder="可选密钥" spellcheck="false" /></label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">加密</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解密</button>
      </div>
      <div class="bifid__output selectable">{{ output }}</div>
      <p class="bifid__hint">Bifid 密码是一种分数加密法，结合替换和置换。I/J 共用。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLO')
const mode = ref('encode')
const key = ref('CRYPTO')

function buildSquare(keyword) {
  const seen = new Set()
  const square = []
  const cleanKey = (keyword || '').toUpperCase().replace(/[^A-Z]/g, '').replace(/J/g, 'I')
  for (const ch of cleanKey) { if (!seen.has(ch)) { seen.add(ch); square.push(ch) } }
  for (let i = 0; i < 26; i++) {
    const ch = String.fromCharCode(65 + i)
    if (ch === 'J') continue
    if (!seen.has(ch)) { seen.add(ch); square.push(ch) }
  }
  return square
}

function getPos(square, ch) {
  const idx = square.indexOf(ch === 'J' ? 'I' : ch)
  return [Math.floor(idx / 5), idx % 5]
}

function encode(text, keyword) {
  const square = buildSquare(keyword)
  const clean = text.toUpperCase().replace(/[^A-Z]/g, '').replace(/J/g, 'I')
  const rows = [], cols = []
  for (const ch of clean) {
    const [r, c] = getPos(square, ch)
    rows.push(r); cols.push(c)
  }
  const combined = [...rows, ...cols]
  let result = ''
  for (let i = 0; i < combined.length; i += 2) {
    const r = combined[i]
    const c = combined[i + 1] ?? 0
    result += square[r * 5 + c]
  }
  return result
}

function decode(text, keyword) {
  const square = buildSquare(keyword)
  const clean = text.toUpperCase().replace(/[^A-Z]/g, '').replace(/J/g, 'I')
  const combined = []
  for (const ch of clean) {
    const [r, c] = getPos(square, ch)
    combined.push(r, c)
  }
  const half = combined.length / 2
  const rows = combined.slice(0, half)
  const cols = combined.slice(half)
  let result = ''
  for (let i = 0; i < rows.length; i++) {
    result += square[rows[i] * 5 + cols[i]]
  }
  return result
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value, key.value) : decode(input.value, key.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.bifid{display:flex;flex-direction:column;gap:12px}
.bifid__field{display:flex;flex-direction:column;gap:4px}
.bifid__field label{font-size:12px;color:var(--text-tertiary)}
.bifid__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.bifid__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.bifid__controls input[type=text]{width:140px;padding:6px 8px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace}
.bifid__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.bifid__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.bifid__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.bifid__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
