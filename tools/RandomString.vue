<template>
  <h-single-layout>
    <div class="random-str">
      <div class="random-str__controls">
        <div class="random-str__field"><label>长度</label><input type="number" v-model.number="length" min="1" max="1000" /></div>
        <div class="random-str__field"><label>数量</label><input type="number" v-model.number="count" min="1" max="100" /></div>
      </div>
      <div class="random-str__charset">
        <label><input type="checkbox" v-model="opts.lower" /> a-z</label>
        <label><input type="checkbox" v-model="opts.upper" /> A-Z</label>
        <label><input type="checkbox" v-model="opts.numbers" /> 0-9</label>
        <label><input type="checkbox" v-model="opts.symbols" /> 符号</label>
      </div>
      <button class="random-str__btn" @click="generate">生成</button>
      <div v-if="results.length" class="random-str__results">
        <div v-for="(s, i) in results" :key="i" class="random-str__result">
          <code>{{ s }}</code>
          <button @click="copy(s)">📋</button>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
const length = ref(32)
const count = ref(5)
const opts = reactive({ lower: true, upper: true, numbers: true, symbols: false })
const results = ref<string[]>([])

function generate(): void {
  let chars = ''
  if (opts.lower) chars += 'abcdefghijklmnopqrstuvwxyz'
  if (opts.upper) chars += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  if (opts.numbers) chars += '0123456789'
  if (opts.symbols) chars += '!@#$%^&*()_+-=[]{}|;:,.<>?'
  if (!chars) { window.$he3?.message.warning('请至少选择一个字符集'); return }
  results.value = Array.from({ length: count.value }, () => {
    const arr = new Uint32Array(length.value)
    crypto.getRandomValues(arr)
    return Array.from(arr).map((n) => chars[n % chars.length]).join('')
  })
}
function copy(s: string): void { window.$he3?.copyText(s); window.$he3?.message.success('已复制') }
generate()
</script>

<style scoped>
.random-str { display: flex; flex-direction: column; gap: 12px; }
.random-str__controls { display: flex; gap: 12px; }
.random-str__field { display: flex; flex-direction: column; gap: 4px; }
.random-str__field label { font-size: 12px; color: var(--text-secondary); }
.random-str__field input { padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-primary); width: 80px; }
.random-str__charset { display: flex; gap: 12px; }
.random-str__charset label { display: flex; align-items: center; gap: 4px; font-size: 13px; color: var(--text-secondary); cursor: pointer; }
.random-str__btn { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.random-str__results { display: flex; flex-direction: column; gap: 6px; }
.random-str__result { display: flex; align-items: center; gap: 8px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); }
.random-str__result code { flex: 1; font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.random-str__result button { border: none; background: transparent; cursor: pointer; font-size: 14px; }
</style>
