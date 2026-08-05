<template>
  <h-single-layout>
    <div class="blake2">
      <div class="blake2__controls">
        <label>变体</label>
        <select v-model="variant">
          <option value="blake2s-256">BLAKE2s-256</option>
          <option value="blake2s-224">BLAKE2s-224</option>
          <option value="blake2s-160">BLAKE2s-160</option>
        </select>
      </div>
      <textarea v-model="input" rows="4" placeholder="输入要计算 BLAKE2 哈希的文本..." spellcheck="false"></textarea>
      <div class="blake2__result">
        <label>BLAKE2s (Hex)</label>
        <div class="blake2__output selectable">{{ hash }}</div>
        <button v-if="hash" @click="copy">复制</button>
      </div>
      <div class="blake2__info">
        BLAKE2s 是为 32 位平台优化的快速加密哈希，比 SHA-256 更快更安全
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const variant = ref('blake2s-256')
const input = ref('Hello BLAKE2!')

// BLAKE2s 纯 JS 实现
const IV = [
  0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a,
  0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19
]

const SIGMA = [
  [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15],
  [14,10,4,8,9,15,13,6,1,12,0,2,11,7,5,3],
  [11,8,12,0,5,2,15,13,10,14,3,6,7,1,9,4],
  [7,9,3,1,13,12,11,14,2,6,5,10,4,0,15,8],
  [9,0,5,7,2,4,10,15,14,1,11,12,6,8,3,13],
  [2,12,6,10,0,11,8,3,4,13,7,5,15,14,1,9],
  [12,5,1,15,14,13,4,10,0,7,6,3,9,2,8,11],
  [13,11,7,14,12,1,3,9,5,0,15,4,8,6,2,10],
  [6,15,14,9,11,3,0,8,12,2,13,7,1,4,10,5],
  [10,2,8,4,7,6,1,5,15,11,9,14,3,12,13,0]
]

function rotl32(x, n) {
  return ((x << n) | (x >>> (32 - n))) >>> 0
}

function G(v, a, b, c, d, x, y) {
  v[a] = (v[a] + v[b] + x) >>> 0
  v[d] = rotl32(v[d] ^ v[a], 16)
  v[c] = (v[c] + v[d]) >>> 0
  v[b] = rotl32(v[b] ^ v[c], 12)
  v[a] = (v[a] + v[b] + y) >>> 0
  v[d] = rotl32(v[d] ^ v[a], 8)
  v[c] = (v[c] + v[d]) >>> 0
  v[b] = rotl32(v[b] ^ v[c], 7)
}

function blake2s(input, outLen) {
  const h = [...IV]
  h[0] ^= 0x01010000 ^ outLen

  const blockLen = 64
  const totalLen = input.length
  const fullBlocks = Math.ceil(totalLen / blockLen)

  for (let i = 0; i < fullBlocks; i++) {
    const block = input.slice(i * blockLen, (i + 1) * blockLen)
    const isLast = (i === fullBlocks - 1)
    const t = isLast ? totalLen : (i + 1) * blockLen

    // Pad block to 64 bytes
    const m = new Array(16).fill(0)
    for (let j = 0; j < block.length; j++) {
      m[(j / 4) | 0] |= block[j] << ((j % 4) * 8)
    }

    const v = [...h, IV[0], IV[1], IV[2], IV[3], IV[4], IV[5], IV[6], IV[7]]
    v[12] ^= t & 0xFFFFFFFF
    v[13] ^= 0
    if (isLast) v[14] ^= 0xFFFFFFFF

    for (let r = 0; r < 10; r++) {
      const s = SIGMA[r]
      G(v, 0, 4, 8, 12, m[s[0]], m[s[1]])
      G(v, 1, 5, 9, 13, m[s[2]], m[s[3]])
      G(v, 2, 6, 10, 14, m[s[4]], m[s[5]])
      G(v, 3, 7, 11, 15, m[s[6]], m[s[7]])
      G(v, 0, 5, 10, 15, m[s[8]], m[s[9]])
      G(v, 1, 6, 11, 12, m[s[10]], m[s[11]])
      G(v, 2, 7, 8, 13, m[s[12]], m[s[13]])
      G(v, 3, 4, 9, 14, m[s[14]], m[s[15]])
    }

    for (let j = 0; j < 8; j++) {
      h[j] ^= v[j] ^ v[j + 8]
      h[j] = h[j] >>> 0
    }
  }

  // 输出前 outLen/4 个 word
  const outWords = Math.ceil(outLen / 4)
  let result = ''
  for (let i = 0; i < outWords; i++) {
    result += ('00000000' + h[i].toString(16)).slice(-8)
  }
  return result.substr(0, outLen * 2)
}

const hash = computed(() => {
  if (!input.value) return ''
  try {
    const outLen = parseInt(variant.value.split('-')[1])
    const bytes = new TextEncoder().encode(input.value)
    return blake2s(bytes, outLen)
  } catch {
    return 'Error'
  }
})

function copy() {
  navigator.clipboard?.writeText(hash.value)
}
</script>

<style scoped>
.blake2 { display: flex; flex-direction: column; gap: 12px; }
.blake2__controls { display: flex; align-items: center; gap: 8px; }
.blake2__controls label { font-size: 13px; color: var(--text-secondary); }
.blake2__controls select { padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; }
.blake2 textarea { width: 100%; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.blake2__result { display: flex; flex-direction: column; gap: 4px; }
.blake2__result label { font-size: 12px; color: var(--text-tertiary); }
.blake2__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.blake2__result button { align-self: flex-start; margin-top: 4px; padding: 6px 16px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 13px; cursor: pointer; }
.blake2__info { padding: 8px 12px; border-radius: 6px; background: var(--bg-base); font-size: 12px; color: var(--text-tertiary); }
</style>
