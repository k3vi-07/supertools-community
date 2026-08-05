<template>
  <h-single-layout>
    <div class="css-grid">
      <div class="css-grid__controls">
        <div class="css-grid__row"><label>列数</label><input type="number" v-model.number="cols" min="1" max="12" /></div>
        <div class="css-grid__row"><label>行数</label><input type="number" v-model.number="rows" min="1" max="12" /></div>
        <div class="css-grid__row"><label>间距</label><input type="text" v-model="gap" placeholder="8px" /></div>
      </div>
      <div class="css-grid__preview" :style="gridStyle">
        <div v-for="i in cols * rows" :key="i" class="css-grid__cell">{{ i }}</div>
      </div>
      <div class="css-grid__output">
        <div class="css-grid__header"><span>CSS</span><button @click="copy">复制</button></div>
        <pre class="css-grid__code selectable">{{ cssCode }}</pre>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const cols = ref(3)
const rows = ref(2)
const gap = ref('12px')
const gridStyle = computed(() => ({ display: 'grid', gridTemplateColumns: `repeat(${cols.value}, 1fr)`, gridTemplateRows: `repeat(${rows.value}, 1fr)`, gap: gap.value }))
const cssCode = computed(() => `display: grid;\ngrid-template-columns: repeat(${cols.value}, 1fr);\ngrid-template-rows: repeat(${rows.value}, 1fr);\ngap: ${gap.value};`)
function copy() { window.$he3?.copyText(cssCode.value); window.$he3?.message.success('已复制') }
</script>

<style scoped>
.css-grid { display: flex; flex-direction: column; gap: 16px; }
.css-grid__controls { display: flex; gap: 16px; }
.css-grid__row { display: flex; flex-direction: column; gap: 4px; }
.css-grid__row label { font-size: 12px; color: var(--text-secondary); }
.css-grid__row input { padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; width: 80px; }
.css-grid__preview { padding: 12px; border: 2px dashed var(--border-color); border-radius: 8px; }
.css-grid__cell { display: flex; align-items: center; justify-content: center; background: var(--color-primary); color: white; border-radius: 4px; padding: 12px; font-weight: 600; font-size: 14px; }
.css-grid__output { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.css-grid__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.css-grid__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.css-grid__code { padding: 12px; font-family: monospace; font-size: 13px; color: var(--color-primary); background: var(--bg-code); }
</style>
