<template>
  <h-single-layout>
    <div class="dec-decode">
      <div class="dec-decode__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入十进制（空格分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="dec-decode__controls">
        <label><input type="checkbox" v-model="space"> 空格分隔</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="dec-decode__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')
const space = ref(true)

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (mode.value === 'encode') {
      const bytes = new TextEncoder().encode(input.value)
      const decs = [...bytes].map(b => String(b))
      return space.value ? decs.join(' ') : decs.join('')
    } else {
      const tokens = input.value.trim().split(/\s+/)
      const bytes = new Uint8Array(tokens.length)
      for (let i = 0; i < tokens.length; i++) {
        const n = parseInt(tokens[i], 10)
        if (isNaN(n) || n < 0 || n > 255) throw new Error(`无效值: ${tokens[i]}`)
        bytes[i] = n
      }
      return new TextDecoder().decode(bytes)
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.dec-decode{display:flex;flex-direction:column;gap:12px}
.dec-decode__field{display:flex;flex-direction:column;gap:4px}
.dec-decode__field label{font-size:12px;color:var(--text-tertiary)}
.dec-decode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.dec-decode__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.dec-decode__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.dec-decode__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.dec-decode__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.dec-decode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
