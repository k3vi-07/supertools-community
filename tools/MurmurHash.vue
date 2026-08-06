<template>
  <h-single-layout>
    <div class="murmur">
      <div class="murmur__field"><label>输入</label><textarea v-model="input" rows="3" spellcheck="false"></textarea></div>
      <div class="murmur__field"><label>种子 (Seed)</label><input type="number" v-model.number="seed" /></div>
      <div class="murmur__grid">
        <div v-for="h in results" :key="h.name" class="murmur__item" @click="copy(h.value)">
          <div class="murmur__name">{{ h.name }}</div>
          <div class="murmur__val selectable">{{ h.value }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const input = ref('Hello MurmurHash!')
const seed = ref(0)

function murmur3_32(key, seed) {
  const bytes = new TextEncoder().encode(key)
  const c1 = 0xcc9e2d51, c2 = 0x1b873593
  let h1 = seed
  const nblocks = Math.floor(bytes.length / 4)
  for (let i = 0; i < nblocks; i++) {
    let k1 = (bytes[i*4]) | (bytes[i*4+1]<<8) | (bytes[i*4+2]<<16) | (bytes[i*4+3]<<24)
    k1 = Math.imul(k1, c1)
    k1 = ((k1 << 15) | (k1 >>> 17))
    k1 = Math.imul(k1, c2)
    h1 ^= k1
    h1 = ((h1 << 13) | (h1 >>> 19))
    h1 = (Math.imul(h1, 5) + 0xe6546b64) | 0
  }
  let k1 = 0
  const tail = bytes.length % 4
  const offset = nblocks * 4
  if (tail >= 3) k1 ^= bytes[offset+2] << 16
  if (tail >= 2) k1 ^= bytes[offset+1] << 8
  if (tail >= 1) {
    k1 ^= bytes[offset]
    k1 = Math.imul(k1, c1)
    k1 = ((k1 << 15) | (k1 >>> 17))
    k1 = Math.imul(k1, c2)
    h1 ^= k1
  }
  h1 ^= bytes.length
  h1 ^= h1 >>> 16
  h1 = Math.imul(h1, 0x85ebca6b)
  h1 ^= h1 >>> 13
  h1 = Math.imul(h1, 0xc2b2ae35)
  h1 ^= h1 >>> 16
  return (h1 >>> 0).toString(16).padStart(8, '0')
}

function murmur2_32(key, seed) {
  const bytes = new TextEncoder().encode(key)
  const m = 0x5bd1e995, r = 24
  let h = (seed ^ bytes.length) | 0
  for (let i = 0; i + 4 <= bytes.length; i += 4) {
    let k = bytes[i] | (bytes[i+1]<<8) | (bytes[i+2]<<16) | (bytes[i+3]<<24)
    k = Math.imul(k, m)
    k ^= k >>> r
    k = Math.imul(k, m)
    h = Math.imul(h, m)
    h ^= k
  }
  const tail = bytes.length % 4
  const offset = bytes.length - tail
  if (tail >= 3) h ^= bytes[offset+2] << 16
  if (tail >= 2) h ^= bytes[offset+1] << 8
  if (tail >= 1) { h ^= bytes[offset]; h = Math.imul(h, m) }
  h ^= h >>> 13
  h = Math.imul(h, m)
  h ^= h >>> 15
  return (h >>> 0).toString(16).padStart(8, '0')
}

const results = computed(() => [
  { name: 'MurmurHash3 (32-bit)', value: murmur3_32(input.value, seed.value >>> 0).toUpperCase() },
  { name: 'MurmurHash2 (32-bit)', value: murmur2_32(input.value, seed.value >>> 0).toUpperCase() },
  { name: 'Murmur3 x86 (Hex)', value: '0x' + murmur3_32(input.value, seed.value >>> 0) },
  { name: 'Murmur3 x86 (Dec)', value: parseInt(murmur3_32(input.value, seed.value >>> 0), 16).toString() },
])

function copy(v) { window.$he3?.copyText(v); window.$he3?.message.success('已复制') }
</script>
<style scoped>
.murmur { display:flex; flex-direction:column; gap:12px; }
.murmur__field { display:flex; flex-direction:column; gap:4px; }
.murmur__field label { font-size:12px; color:var(--text-tertiary); }
.murmur__field input, .murmur__field textarea { padding:8px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); color:var(--text-primary); font-family:monospace; outline:none; resize:vertical; }
.murmur__grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
.murmur__item { padding:10px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); cursor:pointer; transition:all .15s; }
.murmur__item:hover { border-color:var(--color-primary); }
.murmur__name { font-size:12px; color:var(--text-tertiary); margin-bottom:4px; }
.murmur__val { font-size:14px; font-weight:700; color:var(--color-primary); word-break:break-all; }
</style>
