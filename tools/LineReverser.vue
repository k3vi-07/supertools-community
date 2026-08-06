<template>
  <h-single-layout>
    <div class="line-rev">
      <div class="line-rev__controls">
        <label class="line-rev__chk"><input type="radio" value="lines" v-model="mode" /> 每行字符反转</label>
        <label class="line-rev__chk"><input type="radio" value="order" v-model="mode" /> 行顺序反转</label>
        <label class="line-rev__chk"><input type="radio" value="both" v-model="mode" /> 全部反转</label>
        <label class="line-rev__chk"><input type="radio" value="words" v-model="mode" /> 每行单词反转</label>
      </div>
      <div class="line-rev__cols">
        <textarea v-model="input" class="line-rev__textarea" placeholder="输入多行文本..." spellcheck="false"></textarea>
        <div class="line-rev__output selectable">{{ output }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('第一行\n第二行\n第三行')
const mode = ref('lines')

const output = computed(() => {
  const lines = input.value.split('\n')
  let result = lines
  if (mode.value === 'lines' || mode.value === 'both') {
    result = result.map(l => l.split('').reverse().join(''))
  }
  if (mode.value === 'words') {
    result = result.map(l => l.split(/\s+/).reverse().join(' '))
  }
  if (mode.value === 'order' || mode.value === 'both') {
    result = result.reverse()
  }
  return result.join('\n')
})
</script>

<style scoped>
.line-rev { display: flex; flex-direction: column; gap: 12px; }
.line-rev__controls { display: flex; gap: 16px; flex-wrap: wrap; }
.line-rev__chk { font-size: 13px; color: var(--text-secondary); display: flex; align-items: center; gap: 4px; cursor: pointer; }
.line-rev__cols { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.line-rev__textarea { width: 100%; min-height: 200px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; resize: vertical; outline: none; }
.line-rev__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 200px; white-space: pre-wrap; word-break: break-all; overflow: auto; }
</style>
