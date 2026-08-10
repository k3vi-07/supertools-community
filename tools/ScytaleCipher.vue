<template>
  <h-single-layout>
    <div class="scytale">
      <div class="scytale__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入明文' : '输入密文'" spellcheck="false"></textarea>
      </div>
      <div class="scytale__controls">
        <label>缠绕数: <input type="number" v-model.number="turns" min="2" max="50" /></label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">加密</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解密</button>
      </div>
      <div class="scytale__output selectable">{{ output }}</div>
      <p class="scytale__hint">密码棒是古希腊的置换密码，将文字缠绕在棒上读取。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('HELLOWORLD')
const mode = ref('encode')
const turns = ref(3)

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    const text = input.value.replace(/\s/g, '')
    const n = turns.value
    const cols = Math.ceil(text.length / n)
    if (mode.value === 'encode') {
      let result = ''
      for (let c = 0; c < cols; c++) {
        for (let r = 0; r < n; r++) {
          const idx = r * cols + c
          if (idx < text.length) result += text[idx]
        }
      }
      return result
    } else {
      let result = ''
      for (let r = 0; r < n; r++) {
        for (let c = 0; c < cols; c++) {
          const idx = c * n + r
          if (idx < text.length) result += text[idx]
        }
      }
      return result
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.scytale{display:flex;flex-direction:column;gap:12px}
.scytale__field{display:flex;flex-direction:column;gap:4px}
.scytale__field label{font-size:12px;color:var(--text-tertiary)}
.scytale__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.scytale__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.scytale__controls input{width:60px;padding:4px 8px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);color:var(--text-primary)}
.scytale__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.scytale__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.scytale__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.scytale__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
