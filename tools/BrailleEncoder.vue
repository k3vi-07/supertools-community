<template>
  <h-single-layout>
    <div class="braille">
      <div class="braille__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入盲文'" spellcheck="false"></textarea>
      </div>
      <div class="braille__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">文本 -> 盲文</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">盲文 -> 文本</button>
      </div>
      <div class="braille__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')

// Unicode 盲文: U+2800 (⠀) + 点位偏移
// 点位: 1=0x01, 2=0x02, 3=0x04, 4=0x08, 5=0x10, 6=0x20, 7=0x40, 8=0x80
const LETTER_MAP = {
  a: 0x01, b: 0x03, c: 0x09, d: 0x19, e: 0x11, f: 0x0b, g: 0x1b, h: 0x13,
  i: 0x0a, j: 0x1a, k: 0x05, l: 0x07, m: 0x0d, n: 0x1d, o: 0x15, p: 0x0f,
  q: 0x1f, r: 0x17, s: 0x0e, t: 0x1e, u: 0x25, v: 0x27, w: 0x3a, x: 0x2d,
  y: 0x3d, z: 0x35, ' ': 0x00, '.': 0x2c, ',': 0x04, '?': 0x26, '!': 0x24,
  ';': 0x06, ':': 0x16, '-': 0x24, '\'': 0x20, '1': 0x01, '2': 0x03, '3': 0x09,
  '4': 0x19, '5': 0x11, '6': 0x0b, '7': 0x1b, '8': 0x13, '9': 0x0a, '0': 0x1a,
}

const REVERSE_MAP = {}
for (const [k, v] of Object.entries(LETTER_MAP)) REVERSE_MAP[v] = k

const CAPS_PREFIX = 0x28
const NUM_PREFIX = 0x3c

function toBraille(ch) {
  if (ch === ' ') return String.fromCharCode(0x2800)
  if (/[A-Z]/.test(ch)) {
    return String.fromCharCode(0x2800 + CAPS_PREFIX) + String.fromCharCode(0x2800 + (LETTER_MAP[ch.toLowerCase()] || 0))
  }
  if (/[0-9]/.test(ch)) {
    return String.fromCharCode(0x2800 + (LETTER_MAP[ch] || 0))
  }
  return String.fromCharCode(0x2800 + (LETTER_MAP[ch] || 0))
}

function encode(str) {
  return str.split('').map(toBraille).join('')
}

function decode(str) {
  let result = ''
  let caps = false
  for (const ch of str) {
    const dots = ch.charCodeAt(0) - 0x2800
    if (dots < 0 || dots > 255) continue
    if (dots === CAPS_PREFIX) { caps = true; continue }
    const letter = REVERSE_MAP[dots]
    if (letter !== undefined) {
      result += caps ? letter.toUpperCase() : letter
      caps = false
    } else {
      result += ch
    }
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
.braille{display:flex;flex-direction:column;gap:12px}
.braille__field{display:flex;flex-direction:column;gap:4px}
.braille__field label{font-size:12px;color:var(--text-tertiary)}
.braille__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.braille__actions{display:flex;gap:8px}
.braille__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.braille__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.braille__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:18px;min-height:60px;word-break:break-all;line-height:1.8}
</style>
