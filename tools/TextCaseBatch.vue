<template>
  <h-single-layout>
    <div class="text-case">
      <textarea v-model="input" class="text-case__input" placeholder="输入文本..." spellcheck="false"></textarea>
      <div class="text-case__grid">
        <div v-for="c in cases" :key="c.id" class="text-case__card" @click="copy(c.fn(input))">
          <div class="text-case__label">{{ c.name }}</div>
          <div class="text-case__value selectable">{{ c.fn(input) || '—' }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const input = ref('Hello World Foo Bar')

const cases = [
  { id: 'upper', name: 'UPPER CASE', fn: (s) => s.toUpperCase() },
  { id: 'lower', name: 'lower case', fn: (s) => s.toLowerCase() },
  { id: 'title', name: 'Title Case', fn: (s) => s.replace(/\w\S*/g, t => t.charAt(0).toUpperCase() + t.slice(1).toLowerCase()) },
  { id: 'sentence', name: 'Sentence case', fn: (s) => s.charAt(0).toUpperCase() + s.slice(1).toLowerCase() },
  { id: 'camel', name: 'camelCase', fn: (s) => s.toLowerCase().replace(/[^a-zA-Z0-9]+(.)/g, (_, c) => c.toUpperCase()) },
  { id: 'pascal', name: 'PascalCase', fn: (s) => { const c = s.toLowerCase().replace(/[^a-zA-Z0-9]+(.)/g, (_, c) => c.toUpperCase()); return c.charAt(0).toUpperCase() + c.slice(1) } },
  { id: 'snake', name: 'snake_case', fn: (s) => s.replace(/\s+/g, '_').replace(/([A-Z])/g, '_$1').toLowerCase().replace(/^_/, '') },
  { id: 'kebab', name: 'kebab-case', fn: (s) => s.replace(/\s+/g, '-').replace(/([A-Z])/g, '-$1').toLowerCase().replace(/^-/, '') },
  { id: 'dot', name: 'dot.case', fn: (s) => s.replace(/\s+/g, '.').toLowerCase() },
  { id: 'const', name: 'CONST_CASE', fn: (s) => s.replace(/\s+/g, '_').replace(/([A-Z])/g, '_$1').toUpperCase().replace(/^_/, '') },
  { id: 'alt', name: 'aLtErNaTiNg', fn: (s) => s.split('').map((c, i) => i % 2 ? c.toUpperCase() : c.toLowerCase()).join('') },
  { id: 'invert', name: 'iNVERT', fn: (s) => s.split('').map(c => c === c.toUpperCase() ? c.toLowerCase() : c.toUpperCase()).join('') },
]

function copy(text) {
  window.$he3?.copyText(text)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.text-case { display: flex; flex-direction: column; gap: 12px; }
.text-case__input { width: 100%; min-height: 60px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.text-case__grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 8px; }
.text-case__card { padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.text-case__card:hover { border-color: var(--color-primary); }
.text-case__label { font-size: 11px; color: var(--text-tertiary); margin-bottom: 4px; }
.text-case__value { font-size: 13px; color: var(--text-primary); word-break: break-all; max-height: 40px; overflow: hidden; }
</style>
