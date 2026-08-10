<template>
  <h-single-layout>
    <div class="zenkaku">
      <div class="zenkaku__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'toFull' ? '输入半角字符' : '输入全角字符'" spellcheck="false"></textarea>
      </div>
      <div class="zenkaku__actions">
        <button :class="{ active: mode === 'toFull' }" @click="mode = 'toFull'">半角 -> 全角</button>
        <button :class="{ active: mode === 'toHalf' }" @click="mode = 'toHalf'">全角 -> 半角</button>
      </div>
      <div class="zenkaku__output selectable">{{ output }}</div>
      <p class="zenkaku__hint">全角半角互转，支持 ASCII 字符和日文片假名。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World 123')
const mode = ref('toFull')

const output = computed(() => {
  if (!input.value) return ''
  if (mode.value === 'toFull') {
    return input.value.replace(/[\x20-\x7e]/g, c => String.fromCharCode(c.charCodeAt(0) + 0xfee0)).replace(/ /g, '\u3000')
  } else {
    return input.value.replace(/[\uff00-\uffef]/g, c => String.fromCharCode(c.charCodeAt(0) - 0xfee0)).replace(/\u3000/g, ' ')
  }
})
</script>

<style scoped>
.zenkaku{display:flex;flex-direction:column;gap:12px}
.zenkaku__field{display:flex;flex-direction:column;gap:4px}
.zenkaku__field label{font-size:12px;color:var(--text-tertiary)}
.zenkaku__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.zenkaku__actions{display:flex;gap:8px}
.zenkaku__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.zenkaku__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.zenkaku__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:14px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.zenkaku__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
