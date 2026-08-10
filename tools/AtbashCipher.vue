<template>
  <h-single-layout>
    <div class="atbash">
      <div class="atbash__field">
        <label>输入文本</label>
        <textarea v-model="input" rows="5" placeholder="输入文本进行 Atbash 转换..." spellcheck="false"></textarea>
      </div>
      <div class="atbash__output selectable">{{ output }}</div>
      <p class="atbash__hint">Atbash 是一种古老的替换密码，A↔Z、B↔Y、C↔X...编码和解码使用同一操作。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')

const output = computed(() => {
  return input.value.replace(/[a-zA-Z]/g, c => {
    const base = c <= 'Z' ? 65 : 97
    return String.fromCharCode(base + 25 - (c.charCodeAt(0) - base))
  })
})
</script>

<style scoped>
.atbash{display:flex;flex-direction:column;gap:12px}
.atbash__field{display:flex;flex-direction:column;gap:4px}
.atbash__field label{font-size:12px;color:var(--text-tertiary)}
.atbash__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.atbash__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:80px;white-space:pre-wrap;word-break:break-all}
.atbash__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
