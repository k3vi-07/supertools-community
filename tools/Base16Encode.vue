<template>
  <h-single-layout>
    <div class="base16">
      <div class="base16__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Base16 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="base16__controls">
        <label><input type="checkbox" v-model="upper"> 大写</label>
        <label><input type="checkbox" v-model="prefix"> 0x 前缀</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="base16__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')
const mode = ref('encode')
const upper = ref(true)
const prefix = ref(false)

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (mode.value === 'encode') {
      const bytes = new TextEncoder().encode(input.value)
      let hex = [...bytes].map(b => b.toString(16).padStart(2, '0')).join(upper.value ? ' ' : ' ')
      hex = upper.value ? hex.toUpperCase() : hex.toLowerCase()
      return prefix.value ? '0x ' + hex : hex
    } else {
      const clean = input.value.replace(/0x/gi, '').replace(/\s/g, '')
      const bytes = new Uint8Array(clean.length / 2)
      for (let i = 0; i < bytes.length; i++) bytes[i] = parseInt(clean.substr(i * 2, 2), 16)
      return new TextDecoder().decode(bytes)
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.base16{display:flex;flex-direction:column;gap:12px}
.base16__field{display:flex;flex-direction:column;gap:4px}
.base16__field label{font-size:12px;color:var(--text-tertiary)}
.base16__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.base16__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.base16__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.base16__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.base16__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.base16__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
