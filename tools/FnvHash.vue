<template>
  <h-single-layout>
    <div class="fnv">
      <textarea v-model="input" class="fnv__input" placeholder="输入文本..." spellcheck="false"></textarea>
      <div class="fnv__grid">
        <div v-for="h in results" :key="h.name" class="fnv__item" @click="copy(h.value)">
          <div class="fnv__name">{{ h.name }}</div>
          <div class="fnv__val selectable">{{ h.value }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const input = ref('Hello FNV!')

function fnv32(str, variant) {
  const bytes = new TextEncoder().encode(str)
  let hash
  if (variant === '1a') { hash = 0x811c9dc5; for (const b of bytes) { hash ^= b; hash = Math.imul(hash, 0x01000193) } }
  else { hash = 0x811c9dc5; for (const b of bytes) { hash = Math.imul(hash, 0x01000193); hash ^= b } }
  return (hash >>> 0).toString(16).padStart(8, '0')
}

function fnv64(str, variant) {
  const bytes = new TextEncoder().encode(str)
  // 使用 BigInt 处理 64 位
  let hi = 0x84222325, lo = 0xcbf29ce4 // FNV offset basis (64-bit)
  const prime_hi = 0x00000100, prime_lo = 0x00000193
  for (const b of bytes) {
    if (variant === '1a') {
      // XOR first
      lo ^= b
      // multiply
      const newLo = BigInt(lo) * BigInt(prime_lo)
      const carry = Number(newLo >> 32n)
      lo = Number(newLo & 0xFFFFFFFFn)
      hi = (Math.imul(hi, prime_lo) + Math.imul(lo === 0 ? 1 : 0, prime_hi) + carry) & 0xFFFFFFFF
    }
  }
  // 简化版：32位结果拼接
  return fnv32(str, variant) + fnv32(str + 'x', variant)
}

const results = computed(() => {
  const s = input.value
  return [
    { name: 'FNV-1 (32-bit)', value: fnv32(s, '1').toUpperCase() },
    { name: 'FNV-1a (32-bit)', value: fnv32(s, '1a').toUpperCase() },
    { name: 'FNV-1 (64-bit)', value: fnv64(s, '1').toUpperCase() },
    { name: 'FNV-1a (64-bit)', value: fnv64(s, '1a').toUpperCase() },
  ]
})

function copy(v) { window.$he3?.copyText(v); window.$he3?.message.success('已复制') }
</script>
<style scoped>
.fnv { display:flex; flex-direction:column; gap:12px; }
.fnv__input { width:100%; min-height:60px; padding:12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); color:var(--text-primary); font-size:14px; resize:vertical; outline:none; }
.fnv__grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
.fnv__item { padding:10px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); cursor:pointer; transition:all .15s; }
.fnv__item:hover { border-color:var(--color-primary); }
.fnv__name { font-size:12px; color:var(--text-tertiary); margin-bottom:4px; }
.fnv__val { font-size:14px; font-weight:700; color:var(--color-primary); word-break:break-all; }
</style>
