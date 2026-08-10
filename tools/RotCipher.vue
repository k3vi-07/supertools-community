<template>
  <h-single-layout>
    <div class="rot13">
      <div class="rot13__field">
        <label>输入文本</label>
        <textarea v-model="input" rows="5" placeholder="输入要进行 ROT 编码的文本..." spellcheck="false"></textarea>
      </div>
      <div class="rot13__controls">
        <label>位移量: <input type="range" min="1" max="25" v-model.number="shift" /> <span class="rot13__shift">{{ shift }}</span></label>
      </div>
      <div class="rot13__output-section">
        <div class="rot13__output selectable">{{ output }}</div>
        <button class="rot13__copy" @click="copy">复制</button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const shift = ref(13)

const output = computed(() => {
  const s = Number(shift.value) || 13
  return input.value.replace(/[a-zA-Z]/g, c => {
    const base = c <= 'Z' ? 65 : 97
    return String.fromCharCode((c.charCodeAt(0) - base + s) % 26 + base)
  })
})

function copy() {
  window.$he3?.copyText(output.value)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.rot13{display:flex;flex-direction:column;gap:12px}
.rot13__field{display:flex;flex-direction:column;gap:4px}
.rot13__field label{font-size:12px;color:var(--text-tertiary)}
.rot13__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.rot13__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary)}
.rot13__controls input[type=range]{width:200px}
.rot13__shift{font-weight:700;color:var(--color-primary);min-width:24px;display:inline-block}
.rot13__output-section{position:relative}
.rot13__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:80px;white-space:pre-wrap;word-break:break-all}
.rot13__copy{position:absolute;top:8px;right:8px;padding:4px 10px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer;font-size:12px}
.rot13__copy:hover{border-color:var(--color-primary);color:var(--color-primary)}
</style>
