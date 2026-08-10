<template>
  <h-single-layout>
    <div class="oct-decode">
      <div class="oct-decode__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入八进制（空格分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="oct-decode__controls">
        <label><input type="checkbox" v-model="space"> 空格分隔</label>
        <label><input type="checkbox" v-model="prefix"> 0 前缀</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="oct-decode__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')
const space = ref(true)
const prefix = ref(false)

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (mode.value === 'encode') {
      const bytes = new TextEncoder().encode(input.value)
      const octs = [...bytes].map(b => b.toString(8).padStart(3, '0'))
      let result = space.value ? octs.join(' ') : octs.join('')
      if (prefix.value) result = space.value ? octs.map(o => '0' + o).join(' ') : octs.map(o => '0' + o).join('')
      return result
    } else {
      const tokens = input.value.trim().split(/\s+/)
      const bytes = new Uint8Array(tokens.length)
      for (let i = 0; i < tokens.length; i++) bytes[i] = parseInt(tokens[i].replace(/^0+/, '') || '0', 8)
      return new TextDecoder().decode(bytes)
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.oct-decode{display:flex;flex-direction:column;gap:12px}
.oct-decode__field{display:flex;flex-direction:column;gap:4px}
.oct-decode__field label{font-size:12px;color:var(--text-tertiary)}
.oct-decode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.oct-decode__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.oct-decode__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.oct-decode__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.oct-decode__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.oct-decode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
