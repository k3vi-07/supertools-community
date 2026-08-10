<template>
  <h-single-layout>
    <div class="bin-decode">
      <div class="bin-decode__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入二进制（空格或无分隔）'" spellcheck="false"></textarea>
      </div>
      <div class="bin-decode__controls">
        <label><input type="checkbox" v-model="space"> 空格分隔</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="bin-decode__output selectable">{{ output }}</div>
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
      const bins = [...bytes].map(b => b.toString(2).padStart(8, '0'))
      return space.value ? bins.join(' ') : bins.join('')
    } else {
      const clean = input.value.replace(/\s/g, '')
      if (clean.length % 8 !== 0) return '❌ 二进制长度必须是 8 的倍数'
      const bytes = new Uint8Array(clean.length / 8)
      for (let i = 0; i < bytes.length; i++) bytes[i] = parseInt(clean.substr(i * 8, 8), 2)
      return new TextDecoder().decode(bytes)
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.bin-decode{display:flex;flex-direction:column;gap:12px}
.bin-decode__field{display:flex;flex-direction:column;gap:4px}
.bin-decode__field label{font-size:12px;color:var(--text-tertiary)}
.bin-decode__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.bin-decode__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.bin-decode__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.bin-decode__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.bin-decode__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.bin-decode__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
