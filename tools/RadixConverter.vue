<template>
  <h-single-layout>
    <div class="int-convert">
      <div class="int-convert__field">
        <label>输入数字</label>
        <input type="text" v-model="input" placeholder="输入数字..." spellcheck="false" />
      </div>
      <div class="int-convert__base">
        <label>输入进制: </label>
        <select v-model="inBase">
          <option :value="2">二进制 (2)</option>
          <option :value="8">八进制 (8)</option>
          <option :value="10">十进制 (10)</option>
          <option :value="16">十六进制 (16)</option>
          <option :value="36">三十六进制 (36)</option>
        </select>
      </div>
      <div class="int-convert__results">
        <div class="int-convert__result" v-for="b in bases" :key="b.base">
          <span class="int-convert__label">{{ b.name }} ({{ b.base }})</span>
          <span class="int-convert__value selectable">{{ b.value }}</span>
          <button class="int-convert__copy" @click="copy(b.value)">复制</button>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('255')
const inBase = ref(10)

const bases = computed(() => {
  const num = parseInt(input.value.trim(), inBase.value)
  if (isNaN(num)) return []
  return [
    { base: 2, name: '二进制', value: num.toString(2) },
    { base: 8, name: '八进制', value: num.toString(8) },
    { base: 10, name: '十进制', value: num.toString(10) },
    { base: 16, name: '十六进制', value: num.toString(16).toUpperCase() },
    { base: 36, name: '三十六进制', value: num.toString(36).toUpperCase() },
  ]
})

function copy(text) { window.$he3?.copyText(text); window.$he3?.message.success('已复制') }
</script>

<style scoped>
.int-convert{display:flex;flex-direction:column;gap:16px}
.int-convert__field{display:flex;flex-direction:column;gap:4px}
.int-convert__field label{font-size:12px;color:var(--text-tertiary)}
.int-convert__field input{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none}
.int-convert__base{display:flex;align-items:center;gap:8px;font-size:13px;color:var(--text-secondary)}
.int-convert__base select{padding:6px 12px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-primary)}
.int-convert__results{display:flex;flex-direction:column;gap:8px}
.int-convert__result{display:flex;align-items:center;gap:12px;padding:10px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface)}
.int-convert__label{font-size:12px;color:var(--text-faint);min-width:90px}
.int-convert__value{font-family:monospace;font-size:14px;color:var(--text-primary);flex:1;word-break:break-all}
.int-convert__copy{padding:4px 10px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-base);color:var(--text-secondary);cursor:pointer;font-size:12px}
.int-convert__copy:hover{border-color:var(--color-primary);color:var(--color-primary)}
</style>
