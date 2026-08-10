<template>
  <h-single-layout>
    <div class="textdiff-table">
      <div class="textdiff-table__field">
        <label>输入文本（ASCII）</label>
        <textarea v-model="input" rows="4" placeholder="输入文本查看 ASCII 码表..." spellcheck="false"></textarea>
      </div>
      <div class="textdiff-table__table">
        <div class="textdiff-table__header">
          <span>字符</span><span>十进制</span><span>十六进制</span><span>八进制</span><span>二进制</span>
        </div>
        <div v-for="(row, i) in rows" :key="i" class="textdiff-table__row">
          <span class="textdiff-table__char">{{ row.char }}</span>
          <span>{{ row.dec }}</span>
          <span>0x{{ row.hex }}</span>
          <span>0{{ row.oct }}</span>
          <span>{{ row.bin }}</span>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello')

const rows = computed(() => {
  return [...input.value].map(ch => {
    const code = ch.codePointAt(0)
    return {
      char: ch === ' ' ? '␣' : ch === '\n' ? '↵' : ch === '\t' ? '⇥' : ch,
      dec: code,
      hex: code.toString(16).toUpperCase().padStart(2, '0'),
      oct: code.toString(8),
      bin: code.toString(2).padStart(8, '0'),
    }
  })
})
</script>

<style scoped>
.textdiff-table{display:flex;flex-direction:column;gap:12px}
.textdiff-table__field{display:flex;flex-direction:column;gap:4px}
.textdiff-table__field label{font-size:12px;color:var(--text-tertiary)}
.textdiff-table__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.textdiff-table__table{border:1px solid var(--border-color);border-radius:8px;overflow:hidden;font-family:monospace;font-size:13px}
.textdiff-table__header{display:grid;grid-template-columns:60px 80px 80px 80px 1fr;background:var(--bg-active);padding:8px 12px;font-weight:600;color:var(--text-secondary)}
.textdiff-table__row{display:grid;grid-template-columns:60px 80px 80px 80px 1fr;padding:6px 12px;border-top:1px solid var(--border-color)}
.textdiff-table__row:hover{background:var(--bg-hover)}
.textdiff-table__char{font-size:16px;color:var(--color-primary)}
</style>
