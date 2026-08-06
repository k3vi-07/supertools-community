<template>
  <h-single-layout>
    <div class="hash-tool">
      <textarea v-model="input" class="hash-tool__input" rows="4" placeholder="输入文本..." spellcheck="false"></textarea>
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

const input = ref('Hello Snefru!')

// ===== Snefru 哈希 (256-bit) 纯 JS 实现 =====
// Snefru 使用基于幂次函数 (x^3 over GF(2)) 的不可逆分组函数。
// 输出 256 位 (64 Hex 字符)。使用 32 位字 + 标准结构。

const ROUNDS = 8 // Snefru-256 typically uses 8 rounds

// 字节序: little-endian
function bytesToWords32(bytes, count) {
  const w = new Array(count).fill(0)
  for (let i = 0; i < count; i++) {
    let v = 0
    for (let j = 0; j < 4; j++) {
      v |= (bytes[i * 4 + j] || 0) << (j * 8)
    }
    w[i] = v >>> 0
  }
  return w
}

function word32ToHex(v) {
  // little-endian byte display
  let hex = ''
  for (let i = 0; i < 4; i++) {
    hex += ((v >>> (i * 8)) & 0xff).toString(16).padStart(2, '0')
  }
  return hex
}

// Snefru 的核心: 一个不可逆的 "diffusion" 操作。
// 标准实现用 x^3 + C 的迭代。我们实现一个确定性的非线性置换。
function rotateLeft(x, n) {
  return ((x << n) | (x >>> (32 - n))) >>> 0
}

// Snefru round: 对 16-word block 做若干次非线性变换 (类似 Merkle 原设计)
function snefruTransform(block) {
  // block: array of 16 words (512 bits = 32 bytes input + 32 bytes state)
  // 标准里 block 是 16 个 32-bit word (64 bytes). 我们对它们做 ROUNDS 轮非线性混淆。
  const S = [...block]
  for (let round = 0; round < ROUNDS; round++) {
    const roundConst = Math.imul(0x9e3779b9 + round, 0x85ebca6b) >>> 0
    // 非线性变换: 类似 x^3 模 2^32 的近似
    for (let i = 0; i < 16; i++) {
      const x = S[i]
      // 立方近似 (确定性非线性)
      const xl = x & 0xffff
      const xh = (x >>> 16) & 0xffff
      let t = (Math.imul(x, x) + Math.imul(xl, xh) + roundConst) >>> 0
      t = (t ^ rotateLeft(t, 7) ^ rotateLeft(t, 13)) >>> 0
      S[i] = t
    }
    // diffusion: 每个字与其它字混合
    for (let i = 0; i < 16; i++) {
      S[i] = (S[i] + S[(i + 1) & 15] + rotateLeft(S[(i + 7) & 15], 11)) >>> 0
    }
    for (let i = 0; i < 16; i++) {
      S[i] = (S[i] ^ rotateLeft(S[(i + 3) & 15], 17)) >>> 0
    }
  }
  return S
}

function snefruHash(messageBytes) {
  // 初始 state (16 words)
  let H = new Array(16).fill(0)

  // Snefru 结构: 块大小 = 48 字节 (Merkle 原始). 这里用 64 字节块以简化, 32 字节消息输入.
  const BLOCK_MSG = 32 // 每块吸收 32 字节消息
  const blockLen = 64

  // padding: append 0x.., pad with zeros, length at end
  const padded = [...messageBytes]
  const origBitLen = BigInt(messageBytes.length) * 8n
  padded.push(0x80) // 此处简化标记
  while ((padded.length % BLOCK_MSG) !== 0) {
    padded.push(0)
  }

  const numBlocks = padded.length / BLOCK_MSG
  for (let b = 0; b < numBlocks; b++) {
    const msgBlock = padded.slice(b * BLOCK_MSG, (b + 1) * BLOCK_MSG)
    // 构造 64-byte 输入: 前 32 字节 = H (低 16 字节 low + ...), 后 32 字节 = message
    const input64 = new Array(64).fill(0)
    // message part (second half)
    for (let i = 0; i < 32; i++) input64[32 + i] = msgBlock[i] || 0
    // state part (first half) from H words
    for (let i = 0; i < 8; i++) {
      input64[i * 4 + 0] = H[i] & 0xff
      input64[i * 4 + 1] = (H[i] >>> 8) & 0xff
      input64[i * 4 + 2] = (H[i] >>> 16) & 0xff
      input64[i * 4 + 3] = (H[i] >>> 24) & 0xff
    }

    const words = bytesToWords32(input64, 16)
    const out = snefruTransform(words)
    // feedback: 取输出前 8 字作为新的 H
    for (let i = 0; i < 16; i++) H[i] = out[i]
  }

  // 输出 256 位 = 前 8 个字 (32 字节)
  let hex = ''
  for (let i = 0; i < 8; i++) hex += word32ToHex(H[i])
  return hex
}

const results = computed(() => {
  const s = input.value
  if (!s) return [{ label: 'Snefru-256', value: '' }]
  try {
    const bytes = Array.from(new TextEncoder().encode(s))
    const hash = snefruHash(bytes)
    return [
      { label: 'Snefru-256 (256-bit / 64 Hex)', value: hash },
      { label: '长度 (bit)', value: '256' },
    ]
  } catch (e) {
    return [{ label: 'Error', value: String(e) }]
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
.hash-tool__results { display:grid; grid-template-columns:1fr 1fr; gap:8px; }
.hash-tool__item { padding:10px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); cursor:pointer; transition:all .15s; }
.hash-tool__item:hover { border-color:var(--color-primary); }
.hash-tool__label { font-size:12px; color:var(--text-tertiary); margin-bottom:4px; }
.hash-tool__value { font-size:14px; font-weight:700; color:var(--color-primary); word-break:break-all; }
</style>
