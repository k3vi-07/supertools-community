<template>
  <h-single-layout>
    <div class="uuid-cmp">
      <div class="uuid-cmp__controls">
        <button class="uuid-cmp__btn" @click="generate">生成 UUID</button>
        <button class="uuid-cmp__btn" @click="generateBatch">生成 10 个</button>
        <div class="uuid-cmp__sep"></div>
        <label class="uuid-cmp__chk"><input type="checkbox" v-model="upper" /> 大写</label>
        <label class="uuid-cmp__chk"><input type="checkbox" v-model="braces" /> 花括号</label>
        <label class="uuid-cmp__chk"><input type="checkbox" v-model="noDash" /> 去横线</label>
      </div>
      <div class="uuid-cmp__results">
        <div v-for="(u, i) in results" :key="i" class="uuid-cmp__row" @click="copy(u)">
          <span class="uuid-cmp__badge">{{ u.version }}</span>
          <code class="uuid-cmp__code">{{ format(u.value) }}</code>
          <span class="uuid-cmp__copy">📋</span>
        </div>
      </div>
      <div v-if="results.length > 1" class="uuid-cmp__info">
        点击任意行复制 · 共 {{ results.length }} 个 UUID
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const upper = ref(false)
const braces = ref(false)
const noDash = ref(false)
const results = ref([])

// UUID v4
function uuidv4() {
  if (crypto.randomUUID) return { value: crypto.randomUUID(), version: 'v4' }
  const buf = new Uint8Array(16)
  crypto.getRandomValues(buf)
  buf[6] = (buf[6] & 0x0f) | 0x40
  buf[8] = (buf[8] & 0x3f) | 0x80
  const hex = [...buf].map(b => b.toString(16).padStart(2, '0'))
  const val = `${hex.slice(0,4).join('')}-${hex.slice(4,6).join('')}-${hex.slice(6,8).join('')}-${hex.slice(8,10).join('')}-${hex.slice(10,16).join('')}`
  return { value: val, version: 'v4' }
}

// Nil UUID
function nilUuid() {
  return { value: '00000000-0000-0000-0000-000000000000', version: 'nil' }
}

function format(val) {
  let v = val
  if (noDash.value) v = v.replace(/-/g, '')
  if (upper.value) v = v.toUpperCase()
  if (braces.value) v = '{' + v + '}'
  return v
}

function generate() {
  results.value = [uuidv4(), nilUuid()]
}

function generateBatch() {
  const batch = []
  for (let i = 0; i < 10; i++) batch.push(uuidv4())
  results.value = batch
}

function copy(u) {
  window.$he3?.copyText(format(u.value))
  window.$he3?.message.success('已复制')
}

generate()
</script>

<style scoped>
.uuid-cmp { display: flex; flex-direction: column; gap: 12px; }
.uuid-cmp__controls { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.uuid-cmp__btn { padding: 6px 14px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); cursor: pointer; }
.uuid-cmp__btn:hover { border-color: var(--color-primary); color: var(--color-primary); }
.uuid-cmp__sep { width: 1px; height: 20px; background: var(--border-color); margin: 0 4px; }
.uuid-cmp__chk { font-size: 13px; color: var(--text-secondary); display: flex; align-items: center; gap: 4px; cursor: pointer; }
.uuid-cmp__results { display: flex; flex-direction: column; gap: 6px; }
.uuid-cmp__row { display: flex; align-items: center; gap: 8px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.uuid-cmp__row:hover { border-color: var(--color-primary); }
.uuid-cmp__badge { font-size: 10px; font-weight: 700; padding: 2px 6px; border-radius: 4px; background: var(--color-primary); color: white; }
.uuid-cmp__code { flex: 1; font-size: 13px; color: var(--text-primary); word-break: break-all; }
.uuid-cmp__copy { font-size: 14px; }
.uuid-cmp__info { font-size: 12px; color: var(--text-tertiary); text-align: center; }
</style>
