<template>
  <h-single-layout>
    <div class="js-esc">
      <div class="js-esc__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" placeholder="输入文本..." spellcheck="false"></textarea>
      </div>
      <div class="js-esc__controls">
        <button :class="{ active: mode === 'escape' }" @click="mode = 'escape'">转义</button>
        <button :class="{ active: mode === 'unescape' }" @click="mode = 'unescape'">反转义</button>
        <label><input type="checkbox" v-model="unicodeOnly"> 仅非 ASCII</label>
      </div>
      <div class="js-esc__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello 世界 🌍\nNewline')
const mode = ref('escape')
const unicodeOnly = ref(false)

const output = computed(() => {
  if (!input.value) return ''
  if (mode.value === 'escape') {
    if (unicodeOnly.value) {
      return input.value.replace(/[^\x00-\x7f]/g, c => {
        const code = c.codePointAt(0)
        return code > 0xffff ? `\\u{${code.toString(16)}}` : `\\u${code.toString(16).padStart(4, '0')}`
      })
    }
    return JSON.stringify(input.value).slice(1, -1)
  } else {
    return input.value
      .replace(/\\u\{([0-9a-fA-F]+)\}/g, (_, h) => String.fromCodePoint(parseInt(h, 16)))
      .replace(/\\u([0-9a-fA-F]{4})/g, (_, h) => String.fromCharCode(parseInt(h, 16)))
      .replace(/\\n/g, '\n').replace(/\\r/g, '\r').replace(/\\t/g, '\t')
      .replace(/\\b/g, '\b').replace(/\\f/g, '\f').replace(/\\v/g, '\v')
      .replace(/\\"/g, '"').replace(/\\'/g, "'").replace(/\\\\/g, '\\')
      .replace(/\\x([0-9a-fA-F]{2})/g, (_, h) => String.fromCharCode(parseInt(h, 16)))
      .replace(/\\0/g, '\0')
  }
})
</script>

<style scoped>
.js-esc{display:flex;flex-direction:column;gap:12px}
.js-esc__field{display:flex;flex-direction:column;gap:4px}
.js-esc__field label{font-size:12px;color:var(--text-tertiary)}
.js-esc__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.js-esc__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.js-esc__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.js-esc__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.js-esc__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.js-esc__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
</style>
