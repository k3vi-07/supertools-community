<template>
  <h-single-layout>
    <div class="figlet">
      <div class="figlet__input"><label>输入文本</label><input v-model="text" placeholder="输入文本..." maxlength="30" /></div>
      <div class="figlet__fonts">
        <button v-for="f in fonts" :key="f.id" :class="{active: font===f.id}" @click="font=f.id">{{ f.label }}</button>
      </div>
      <div class="figlet__output">
        <div class="figlet__header"><span>ASCII Art</span><button v-if="output" @click="copy"><span>📋</span> 复制</button></div>
        <pre class="figlet__art selectable">{{ output }}</pre>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const text = ref('Hello')
const fonts = [
  { id: 'block', label: 'Block' },
  { id: 'banner', label: 'Banner' },
  { id: 'slant', label: 'Slant' },
  { id: 'small', label: 'Small' }
]
const font = ref('block')

// 简化的 ASCII Art 字体映射
const blockChars: Record<string, string[]> = {
  'H': ['█   █','█   █','█████','█   █','█   █'],
  'e': ['█████','█    ','█████','█    ','█████'],
  'l': ['█    ','█    ','█    ','█    ','█████'],
  'o': ['█████','█   █','█   █','█   █','█████'],
  ' ': ['     ','     ','     ','     ','     '],
  'W': ['█   █','█   █','█ █ █','██ ██','█   █'],
  'r': ['████ ','█   █','████ ','█    ','█    '],
  'd': ['   ██','   █ ','   █ ','█  █ ',' ██  '],
  'S': ['█████','█    ','█████','    █','█████'],
  'u': ['█   █','█   █','█   █','█   █','█████'],
  'p': ['█████','█   █','█████','█    ','█    '],
  'T': ['█████','  █  ','  █  ','  █  ','  █  '],
  'i': ['███',' █ ',' █ ',' █ ','███'],
  'A': [' █████','█     █','███████','█     █','█     █'],
  'B': ['█████ ','█    █','█████ ','█    █','█████ '],
  'C': [' █████','█     ','█     ','█     ',' █████']
}

const output = computed(() => {
  const chars = text.value.toUpperCase().split('')
  const rows: string[] = ['', '', '', '', '']
  for (const c of chars) {
    const art = blockChars[c]
    if (art) {
      for (let i = 0; i < 5; i++) rows[i] += art[i] + ' '
    }
  }
  return rows.join('\n')
})

function copy(): void {
  window.$he3?.copyText(output.value)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.figlet { display: flex; flex-direction: column; gap: 12px; }
.figlet__input { display: flex; flex-direction: column; gap: 4px; }
.figlet__input label { font-size: 12px; color: var(--text-secondary); }
.figlet__input input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.figlet__fonts { display: flex; gap: 4px; flex-wrap: wrap; }
.figlet__fonts button { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.figlet__fonts button.active { background: var(--color-primary); color: white; }
.figlet__output { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.figlet__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.figlet__header button { border: none; background: transparent; color: var(--text-secondary); cursor: pointer; font-size: 12px; }
.figlet__art { padding: 16px; font-family: monospace; font-size: 14px; line-height: 1.2; color: var(--color-primary); background: var(--bg-code); overflow-x: auto; }
</style>
