<template>
  <h-single-layout>
    <div class="text-dedup">
      <div class="text-dedup__controls">
        <label class="text-dedup__opt"><input type="checkbox" v-model="trimWhitespace" /> 去除首尾空格</label>
        <label class="text-dedup__opt"><input type="checkbox" v-model="caseSensitive" /> 区分大小写</label>
        <label class="text-dedup__opt"><input type="checkbox" v-model="ignoreEmpty" /> 忽略空行</label>
      </div>
      <div class="text-dedup__io">
        <textarea v-model="input" class="text-dedup__input selectable" placeholder="输入要去重的文本，每行一条..." spellcheck="false"></textarea>
        <div class="text-dedup__output-area">
          <div class="text-dedup__output selectable">{{ output }}</div>
          <button class="text-dedup__copy" @click="copyOutput">复制结果</button>
        </div>
      </div>
      <div class="text-dedup__stats">
        <span>原始: {{ originalCount }} 行</span>
        <span>去重后: {{ outputCount }} 行</span>
        <span>删除: {{ originalCount - outputCount }} 行</span>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('apple\nbanana\napple\ncherry\nbanana\ndate\napple\nelderberry')
const trimWhitespace = ref(true)
const caseSensitive = ref(true)
const ignoreEmpty = ref(true)

const lines = computed(() => input.value.split('\n'))

const processedLines = computed(() => {
  let arr = lines.value
  if (trimWhitespace.value) arr = arr.map(l => l.trim())
  if (ignoreEmpty.value) arr = arr.filter(l => l !== '')
  if (!caseSensitive.value) {
    const seen = new Set()
    const result = []
    const origArr = trimWhitespace.value ? lines.value.map(l => l.trim()) : lines.value
    for (let i = 0; i < origArr.length; i++) {
      const processed = trimWhitespace.value ? origArr[i].trim() : origArr[i]
      if (ignoreEmpty.value && processed === '') continue
      const key = processed.toLowerCase()
      if (!seen.has(key)) {
        seen.add(key)
        result.push(processed)
      }
    }
    return result
  }
  return [...new Set(arr)]
})

const originalCount = computed(() => {
  let arr = lines.value
  if (trimWhitespace.value) arr = arr.map(l => l.trim())
  if (ignoreEmpty.value) arr = arr.filter(l => l !== '')
  return arr.length
})

const output = computed(() => processedLines.value.join('\n'))
const outputCount = computed(() => processedLines.value.length)

function copyOutput() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.text-dedup { display: flex; flex-direction: column; gap: 12px; }
.text-dedup__controls { display: flex; gap: 16px; }
.text-dedup__opt { display: flex; align-items: center; gap: 6px; font-size: 13px; color: var(--text-secondary); cursor: pointer; }
.text-dedup__io { display: flex; gap: 12px; }
.text-dedup__input { flex: 1; min-height: 200px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.text-dedup__output-area { flex: 1; display: flex; flex-direction: column; gap: 8px; }
.text-dedup__output { flex: 1; min-height: 180px; padding: 12px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); white-space: pre-wrap; overflow-y: auto; }
.text-dedup__copy { padding: 8px 16px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 13px; cursor: pointer; }
.text-dedup__stats { display: flex; gap: 20px; font-size: 13px; color: var(--text-tertiary); }
</style>
