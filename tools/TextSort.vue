<template>
  <h-single-layout>
    <div class="text-sort">
      <div class="text-sort__controls">
        <button v-for="m in modes" :key="m.id" :class="{ active: mode === m.id }" @click="mode = m.id">{{ m.label }}</button>
        <label class="text-sort__opt"><input type="checkbox" v-model="ignoreCase" /> 忽略大小写</label>
      </div>
      <div class="text-sort__io">
        <textarea v-model="input" class="text-sort__input selectable" placeholder="输入要排序的文本..." spellcheck="false"></textarea>
        <div class="text-sort__output selectable">{{ output }}</div>
      </div>
      <div class="text-sort__stats">
        <span>{{ lineCount }} 行</span>
        <button class="text-sort__copy" @click="copyOutput">复制</button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('banana\nApple\ncherry\napple\nBanana\nDate\n3line\n1line\n10line')
const mode = ref('asc')
const ignoreCase = ref(false)

const modes = [
  { id: 'asc', label: '升序 A→Z' },
  { id: 'desc', label: '降序 Z→A' },
  { id: 'natural', label: '自然排序' },
  { id: 'length', label: '按长度' },
  { id: 'shuffle', label: '随机打乱' },
  { id: 'reverse', label: '反转行序' }
]

const lines = computed(() => input.value.split('\n').filter(l => l !== ''))

function naturalCompare(a, b) {
  return a.localeCompare(b, undefined, { numeric: true, sensitivity: 'base' })
}

const output = computed(() => {
  let arr = [...lines.value]
  const cmp = ignoreCase.value
    ? (a, b) => a.toLowerCase().localeCompare(b.toLowerCase())
    : (a, b) => a.localeCompare(b)

  switch (mode.value) {
    case 'asc': arr.sort(cmp); break
    case 'desc': arr.sort((a, b) => cmp(b, a)); break
    case 'natural': arr.sort((a, b) => naturalCompare(a, b)); break
    case 'length': arr.sort((a, b) => a.length - b.length || cmp(a, b)); break
    case 'shuffle':
      for (let i = arr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[arr[i], arr[j]] = [arr[j], arr[i]]
      }
      break
    case 'reverse': arr.reverse(); break
  }
  return arr.join('\n')
})

const lineCount = computed(() => lines.value.length)

function copyOutput() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.text-sort { display: flex; flex-direction: column; gap: 12px; }
.text-sort__controls { display: flex; gap: 4px; flex-wrap: wrap; align-items: center; }
.text-sort__controls button { padding: 6px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.text-sort__controls button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.text-sort__opt { display: flex; align-items: center; gap: 6px; font-size: 12px; color: var(--text-secondary); margin-left: 8px; cursor: pointer; }
.text-sort__io { display: flex; gap: 12px; }
.text-sort__input { flex: 1; min-height: 200px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.text-sort__output { flex: 1; min-height: 200px; padding: 12px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); white-space: pre-wrap; overflow-y: auto; }
.text-sort__stats { display: flex; align-items: center; justify-content: space-between; }
.text-sort__stats span { font-size: 13px; color: var(--text-tertiary); }
.text-sort__copy { padding: 6px 16px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 13px; cursor: pointer; }
</style>
