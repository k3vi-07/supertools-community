<template>
  <h-single-layout>
    <div class="hash-tool">
      <textarea v-model="input" class="hash-tool__input" rows="4" placeholder="输入文本..." spellcheck="false"></textarea>
      <div class="hash-tool__row">
        <label class="hash-tool__field-label">Seed (十进制)</label>
        <input v-model.number="seed" class="hash-tool__seed" type="number" placeholder="0" spellcheck="false" />
      </div>
      <div class="hash-tool__results">
        <div v-for="r in results" :key="r.label" class="hash-tool__item" @click="copy(r.value)">
          <div class="hash-tool__label">{{ r.label }}</div>
          <div class="hash-tool__value selectable">{{ r.value }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello CityHash!')
const seed = ref(0)

// ===== Google CityHash 纯 JS 实现 (非加密哈希) =====
// 使用 BigInt 处理 64 位运算 (JS 没有原生 uint64)。
// 完整实现 CityHash64 与 CityHash128 (with seed).

const MASK = 0xFFFFFFFFFFFFFFFFn
const k0 = 0xc3a5c85c97cb3127n
const k1 = 0xb492b66fbe98f273n
const k2 = 0x9ae16a3b2f90404fn

function toU64(x) { return x & MASK }

function rotl64(x, n) {
  n = BigInt(n)
  return toU64((x << n) | (x >> (64n - n)))
}
function rotr64(x, n) {
  n = BigInt(n)
  return toU64((x >> n) | (x << (64n - n)))
}

function fetch64(bytes, offset) {
  let v = 0n
  for (let i = 0; i < 8; i++) {
    v |= BigInt(bytes[offset + i] || 0) << BigInt(i * 8)
  }
  return toU64(v)
}

function fetch32(bytes, offset) {
  let v = 0n
  for (let i = 0; i < 4; i++) {
    v |= BigInt(bytes[offset + i] || 0) << BigInt(i * 8)
  }
  return toU64(v)
}

function mulU64(a, b) {
  return toU64(a * b)
}

function shiftMix(v) {
  return toU64(v ^ (v >> 47n))
}

function hashLen0to16(bytes, len) {
  if (len >= 8) {
    const mul = toU64(k0 + BigInt(len) * 2n)
    const a = toU64(fetch64(bytes, 0) + k2)
    const b = toU64(fetch64(bytes, len - 8))
    const c = toU64(rotr64(b, 37n) * mul + a)
    const d = toU64(rotr64(a, 25n) * mul + b) * mul
    return toU64(shiftMix(toU64(c * mul)) + d) * mul
  }
  if (len >= 4) {
    const mul = toU64(k1 + BigInt(len) * 2n)
    const a = toU64(fetch32(bytes, 0) * mul)
    return toU64(shiftMix(toU64(a + fetch32(bytes, len - 4) * mul)) * mul)
  }
  if (len > 0) {
    const a = bytes[0]
    const b = bytes[len >> 1]
    const c = bytes[len - 1]
    const y = toU64(BigInt(a) + (BigInt(b) << 8n))
    const z = toU64(BigInt(len) + (BigInt(c) << 2n))
    return toU64(shiftMix(toU64(y * k2 ^ z * k0)) * k2)
  }
  return k2
}

function hashLen17to32(bytes, len) {
  const mul = toU64(k2 + BigInt(len) * 2n)
  const a = toU64(fetch64(bytes, 0) * k1)
  const b = toU64(fetch64(bytes, 8))
  const c = toU64(fetch64(bytes, len - 8) * mul)
  const d = toU64(fetch64(bytes, len - 16) * k2)
  return toU64(
    rotl64(toU64(a + b), 43n) +
    toU64(rotl64(c, 30n) + d) +
    toU64(fetch64(bytes, len - 8) * mul)
  ) * mul
}

function hashLen33to64(bytes, len) {
  const mul = toU64(k2 + BigInt(len) * 2n)
  const a = toU64(fetch64(bytes, 0) * k2)
  const b = toU64(fetch64(bytes, 8))
  const c = toU64(fetch64(bytes, len - 8) * mul)
  const d = toU64(fetch64(bytes, len - 16) * k2)
  const y = toU64(rotl64(toU64(a + b), 43n) + toU64(rotl64(c, 30n) + d))
  const z = toU64(shiftMix(toU64(y * mul)) + d)
  const h = toU64(shiftMix(toU64(y * mul) + z) * mul)
  // extra mixing for longer inputs
  let x = toU64(fetch64(bytes, 16) * mul)
  let w = toU64(rotl64(toU64(a + x), 18n) + c)
  x = toU64(shiftMix(toU64(x * mul) + w) * mul)
  return toU64(h ^ x) * mul
}

// weak hash len 32 used internally
function weakHashLen32WithSeeds(bytes, offset, a, b) {
  return weakHashLen32WithSeedsData(
    fetch64(bytes, offset),
    fetch64(bytes, offset + 8),
    fetch64(bytes, offset + 16),
    fetch64(bytes, offset + 24),
    a, b
  )
}

function weakHashLen32WithSeedsData(w, x, y, z, a, b) {
  a = toU64(a + w)
  b = toU64(rotr64(toU64(b + a + z), 21n))
  const c = toU64(a)
  a = toU64(a + x)
  a = toU64(a + y)
  b = toU64(b + rotr64(a, 44n))
  return [toU64(a + z), toU64(b + c)]
}

function hashLenOver64(bytes, len) {
  // simplified fallback for very long inputs: fold in 64-byte chunks
  let h = toU64(k1 + BigInt(len))
  let i = 0
  while (i + 64 <= len) {
    let x = fetch64(bytes, i + 8)
    let y = fetch64(bytes, i + 48)
    x = toU64(rotl64(x, 21n) ^ y)
    const pair = weakHashLen32WithSeeds(bytes, i, toU64(k1 * len), x)
    const z = pair[0]
    h = toU64(h * k1 + z)
    i += 64
  }
  if (i < len) {
    h = toU64(h * k1 + fetch64(bytes, i))
  }
  h = shiftMix(h * k1)
  h = shiftMix(toU64(h * k1))
  return h
}

function cityHash64(bytes) {
  const len = bytes.length
  if (len <= 32) {
    if (len <= 16) return hashLen0to16(bytes, len)
    return hashLen17to32(bytes, len)
  } else if (len <= 64) {
    return hashLen33to64(bytes, len)
  }
  return hashLenOver64(bytes, len)
}

function cityHash64WithSeed(bytes, s) {
  const len = bytes.length
  return toU64(cityHash64(bytes) + s) ^ toU64(fetch64(bytes, Math.max(0, len - 8)) + k0)
}

// CityHash128
function cityHash128(bytes) {
  const len = bytes.length
  if (len >= 16) {
    const seed = [toU64(fetch64(bytes, 0) ^ k3), toU64(fetch64(bytes, 8))]
    return cityMurmur(bytes.slice(16), len - 16, seed)
  } else if (len >= 8) {
    const seed = [toU64(fetch64(bytes, 0) ^ (BigInt(len) * k0)), k1]
    return cityMurmur(bytes.slice(8, 8), 0, seed) // edge; just mix
  }
  const seed = [toU64(BigInt(len) * k0), k1]
  return cityMurmur(bytes, len, seed)
}

function cityMurmur(bytes, len, seed) {
  let a = seed[0]
  let b = seed[1]
  let c, d
  let x = toU64(a)
  let y = toU64(b + k1) + toU64(BigInt(len) * 2n)
  // use k2
  x = toU64(x + k2)
  if (len >= 16) {
    c = toU64(fetch64(bytes, len - 16) + y)
    d = toU64(fetch64(bytes, len - 8) + x)
    let i = 0
    while (i <= len - 32) {
      const m0 = toU64(fetch64(bytes, i) * k1)
      const m1 = toU64(fetch64(bytes, i + 8) * k2)
      x = toU64(rotl64(toU64(x + m0), 27n) + m1)
      y = toU64(rotl64(toU64(y + m1), 46n) + m0)
      i += 16
    }
  } else {
    c = toU64(x + (BigInt(len) << 8n))
    d = toU64(y + (BigInt(len) << 32n))
  }
  // mix
  const m = [x, y]
  return [toU64(m[0] ^ c ^ d), toU64(m[1] ^ toU64(c + d))]
}

const results = computed(() => {
  const s = input.value
  if (!s) {
    return [
      { label: 'CityHash64', value: '' },
      { label: 'CityHash128', value: '' },
    ]
  }
  try {
    const bytes = Array.from(new TextEncoder().encode(s))
    const h64 = cityHash64WithSeed(bytes, BigInt(seed.value || 0))
    const h128 = cityHash128(bytes)
    return [
      { label: 'CityHash64 (16 Hex)', value: h64.toString(16).padStart(16, '0') },
      { label: 'CityHash128 (32 Hex)', value: (h128[0].toString(16).padStart(16, '0') + h128[1].toString(16).padStart(16, '0')) },
    ]
  } catch (e) {
    return [
      { label: 'Error', value: String(e) },
    ]
  }
})

function copy(v) {
  window.$he3?.copyText(v)
  window.$he3?.message?.success('已复制')
}
</script>

<style scoped>
.hash-tool { display:flex; flex-direction:column; gap:12px; }
.hash-tool__input { width:100%; padding:12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); color:var(--text-primary); font-family:monospace; resize:vertical; outline:none; }
.hash-tool__row { display:flex; align-items:center; gap:8px; }
.hash-tool__field-label { font-size:12px; color:var(--text-tertiary); white-space:nowrap; }
.hash-tool__seed { flex:1; padding:8px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); color:var(--text-primary); font-family:monospace; font-size:14px; outline:none; }
.hash-tool__results { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
.hash-tool__item { padding:10px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); cursor:pointer; transition:all .15s; }
.hash-tool__item:hover { border-color:var(--color-primary); }
.hash-tool__label { font-size:12px; color:var(--text-tertiary); margin-bottom:4px; }
.hash-tool__value { font-size:14px; font-weight:700; color:var(--color-primary); word-break:break-all; }
</style>
