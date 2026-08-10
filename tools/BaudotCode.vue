<template>
  <h-single-layout>
    <div class="baudot">
      <div class="baudot__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Baudot 码（空格分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="baudot__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> Baudot</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">Baudot -> 文本</button>
      </div>
      <div class="baudot__output selectable">{{ output }}</div>
      <p class="baudot__hint">Baudot 码是早期电报使用的 5 位编码，ITA2 国际电报字母表标准。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLO')
const mode = ref('encode')

const LTRS = '\u0000E\nA SIU\rDRJNFCKTZLWHYPQOBG VX\u0000MXV\u0000'
const FIGS = '\u00003\n\u0007 \u0004\'\r,!:?+#\u0000()./\u0000$=&\u0000"@\u0000\u0000%\u0000\u0000\u0000 \u0000\u0000\u0000'

function encode(str) {
  let result = []
  let figsMode = false
  for (const ch of str.toUpperCase()) {
    let idx
    if (/[A-Z]/.test(ch)) {
      if (figsMode) { result.push('11011'); figsMode = false }
      idx = LTRS.indexOf(ch)
    } else if (ch === ' ') {
      if (figsMode) { result.push('11011'); figsMode = false }
      idx = 4
    } else {
      idx = FIGS.indexOf(ch)
      if (idx >= 0 && !figsMode) { result.push('11011'); figsMode = true }
    }
    if (idx < 0) continue
    result.push(idx.toString(2).padStart(5, '0'))
  }
  return result.join(' ')
}

function decode(str) {
  let result = ''
  let figsMode = false
  for (const code of str.trim().split(/\s+/)) {
    if (code.length !== 5) continue
    const idx = parseInt(code, 2)
    if (idx === 27) { figsMode = true; continue }
    if (idx === 31) { figsMode = false; continue }
    result += figsMode ? (FIGS[idx] || '') : (LTRS[idx] || '')
  }
  return result
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value) : decode(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.baudot{display:flex;flex-direction:column;gap:12px}
.baudot__field{display:flex;flex-direction:column;gap:4px}
.baudot__field label{font-size:12px;color:var(--text-tertiary)}
.baudot__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.baudot__actions{display:flex;gap:8px}
.baudot__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.baudot__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.baudot__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.baudot__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
