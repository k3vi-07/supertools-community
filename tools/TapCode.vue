<template>
  <h-single-layout>
    <div class="tapcode">
      <div class="tapcode__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入敲击码（用空格分隔数字对）'" spellcheck="false"></textarea>
      </div>
      <div class="tapcode__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> 敲击码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">敲击码 -> 文本</button>
      </div>
      <div class="tapcode__output selectable">{{ output }}</div>
      <p class="tapcode__hint">敲击码（Tap Code）将字母分为 5×5 网格，K 和 C 共用。用两个数字表示行列位置。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLO')
const mode = ref('encode')

// 5x5 网格 (K/C 共用)
const GRID = [
  ['A','B','C','D','E'],
  ['F','G','H','I','J'],
  ['L','M','N','O','P'],
  ['Q','R','S','T','U'],
  ['V','W','X','Y','Z'],
]

function findPos(ch) {
  const c = ch === 'K' ? 'C' : ch
  for (let r = 0; r < 5; r++) {
    for (let col = 0; col < 5; col++) {
      if (GRID[r][col] === c) return [r + 1, col + 1]
    }
  }
  return null
}

function encode(str) {
  return str.toUpperCase().split('').map(ch => {
    if (ch === ' ') return '0 0'
    const pos = findPos(ch)
    return pos ? pos.join(' ') : ''
  }).filter(Boolean).join('  ')
}

function decode(str) {
  return str.trim().split(/\s{2,}/).map(pair => {
    const [r, c] = pair.trim().split(/\s+/).map(Number)
    if (!r || !c) return ' '
    if (r < 1 || r > 5 || c < 1 || c > 5) return ''
    return GRID[r - 1][c - 1]
  }).join('')
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value) : decode(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.tapcode{display:flex;flex-direction:column;gap:12px}
.tapcode__field{display:flex;flex-direction:column;gap:4px}
.tapcode__field label{font-size:12px;color:var(--text-tertiary)}
.tapcode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.tapcode__actions{display:flex;gap:8px}
.tapcode__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.tapcode__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.tapcode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.tapcode__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
