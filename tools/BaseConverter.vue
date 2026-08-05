<template>
  <h-single-layout>
    <div class="converter">
      <div class="converter__type">
        <button v-for="t in types" :key="t.id" :class="{active: type===t.id}" @click="type=t.id">{{ t.label }}</button>
      </div>
      <div class="converter__row">
        <input v-model="input" :placeholder="types.find(t=>t.id===type)?.label + '...'" @input="convert" />
        <span class="converter__arrow">→</span>
        <div class="converter__output selectable">{{ output }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const types = [
  { id: 'bin2dec', label: '二进制→十进制' },
  { id: 'dec2bin', label: '十进制→二进制' },
  { id: 'hex2dec', label: '十六进制→十进制' },
  { id: 'dec2hex', label: '十进制→十六进制' },
  { id: 'bin2hex', label: '二进制→十六进制' },
  { id: 'oct2dec', label: '八进制→十进制' }
]
const type = ref('bin2dec')
const input = ref('1010')
const output = ref('10')

function convert() {
  const v = input.value.trim()
  try {
    switch (type.value) {
      case 'bin2dec': output.value = parseInt(v, 2).toString(10); break
      case 'dec2bin': output.value = parseInt(v, 10).toString(2); break
      case 'hex2dec': output.value = parseInt(v, 16).toString(10); break
      case 'dec2hex': output.value = parseInt(v, 10).toString(16).toUpperCase(); break
      case 'bin2hex': output.value = parseInt(v, 2).toString(16).toUpperCase(); break
      case 'oct2dec': output.value = parseInt(v, 8).toString(10); break
    }
  } catch { output.value = '错误' }
}
</script>

<style scoped>
.converter { display: flex; flex-direction: column; gap: 16px; }
.converter__type { display: flex; gap: 4px; flex-wrap: wrap; }
.converter__type button { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.converter__type button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.converter__row { display: flex; align-items: center; gap: 12px; }
.converter__row input { flex: 1; padding: 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 16px; outline: none; }
.converter__arrow { font-size: 20px; color: var(--text-tertiary); }
.converter__output { flex: 1; padding: 10px; border: 1px solid var(--color-primary); border-radius: 6px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 16px; font-weight: 600; color: var(--color-primary); }
</style>
