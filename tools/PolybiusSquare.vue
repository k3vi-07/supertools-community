<template>
  <h-single-layout>
    <div class="polybius">
      <div class="polybius__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Polybius 码（空格分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="polybius__controls">
        <label>密钥: <input type="text" v-model="key" placeholder="可选密钥" spellcheck="false" /></label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="polybius__output selectable">{{ output }}</div>
      <p class="polybius__hint">波利比奥斯方阵：5×5 网格，I/J 共用，每字母用两位数字表示。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLO')
const mode = ref('encode')
const key = ref('')

function buildSquare(keyword) {
  const seen = new Set()
  const square = []
  const clean = (keyword || '').toUpperCase().replace(/[^A-Z]/g, '').replace(/J/g, 'I')
  for (const ch of clean) { if (!seen.has(ch)) { seen.add(ch); square.push(ch) } }
  for (let i = 0; i < 26; i++) {
    const ch = String.fromCharCode(65 + i)
    if (ch === 'J') continue
    if (!seen.has(ch)) { seen.add(ch); square.push(ch) }
  }
  return square
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    const square = buildSquare(key.value)
    if (mode.value === 'encode') {
      return input.value.toUpperCase().replace(/J/g, 'I').split('').map(ch => {
        const idx = square.indexOf(ch)
        if (idx < 0) return ''
        return String(Math.floor(idx / 5) + 1) + String((idx % 5) + 1)
      }).filter(Boolean).join(' ')
    } else {
      return input.value.trim().split(/\s+/).map(pair => {
        const r = parseInt(pair[0]) - 1
        const c = parseInt(pair[1]) - 1
        if (r < 0 || r > 4 || c < 0 || c > 4) return ''
        return square[r * 5 + c]
      }).join('')
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.polybius{display:flex;flex-direction:column;gap:12px}
.polybius__field{display:flex;flex-direction:column;gap:4px}
.polybius__field label{font-size:12px;color:var(--text-tertiary)}
.polybius__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.polybius__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.polybius__controls input[type=text]{width:140px;padding:6px 8px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace}
.polybius__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.polybius__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.polybius__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.polybius__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
