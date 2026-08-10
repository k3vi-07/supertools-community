<template>
  <h-single-layout>
    <div class="nato">
      <div class="nato__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 NATO 字母（空格分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="nato__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> NATO</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">NATO -> 文本</button>
      </div>
      <div class="nato__output selectable">{{ output }}</div>
      <p class="nato__hint">北约音标字母表（NATO phonetic alphabet），航空和军事通信标准。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')

const MAP = {
  A: 'Alpha', B: 'Bravo', C: 'Charlie', D: 'Delta', E: 'Echo', F: 'Foxtrot',
  G: 'Golf', H: 'Hotel', I: 'India', J: 'Juliet', K: 'Kilo', L: 'Lima',
  M: 'Mike', N: 'November', O: 'Oscar', P: 'Papa', Q: 'Quebec', R: 'Romeo',
  S: 'Sierra', T: 'Tango', U: 'Uniform', V: 'Victor', W: 'Whiskey', X: 'X-ray',
  Y: 'Yankee', Z: 'Zulu', '0': 'Zero', '1': 'One', '2': 'Two', '3': 'Three',
  '4': 'Four', '5': 'Five', '6': 'Six', '7': 'Seven', '8': 'Eight', '9': 'Niner',
  ' ': '(空格)',
}

const REVERSE = {}
for (const [k, v] of Object.entries(MAP)) REVERSE[v.toLowerCase()] = k

function encode(str) {
  return str.toUpperCase().split('').map(ch => MAP[ch] || ch).join(' ')
}

function decode(str) {
  return str.split(/\s+/).map(word => {
    const w = word.replace(/[()（）]/g, '').toLowerCase()
    if (w === '空格' || w === 'space') return ' '
    return REVERSE[w] || ''
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
.nato{display:flex;flex-direction:column;gap:12px}
.nato__field{display:flex;flex-direction:column;gap:4px}
.nato__field label{font-size:12px;color:var(--text-tertiary)}
.nato__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.nato__actions{display:flex;gap:8px}
.nato__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.nato__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.nato__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.nato__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
