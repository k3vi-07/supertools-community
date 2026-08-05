<template>
  <h-single-layout>
    <div class="sm3-hash">
      <textarea v-model="input" rows="4" placeholder="输入要计算 SM3 哈希的文本..." spellcheck="false"></textarea>
      <div class="sm3-hash__result">
        <label>SM3 哈希 (Hex)</label>
        <div class="sm3-hash__output selectable">{{ hash }}</div>
        <button v-if="hash" @click="copy">复制</button>
      </div>
      <div class="sm3-hash__info">
        <span>SM3 是中国国家密码管理局发布的密码杂凑算法标准（GM/T 0004-2012），输出 256 位</span>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello SM3 国密哈希')

// SM3 纯 JS 实现
const IV = [0x7380166f, 0x4914b2b9, 0x172442d7, 0xda8a0600, 0xa96f30bc, 0x163138aa, 0xe38dee4d, 0xb0fb0e4e]

function rotl(x, n) {
  return ((x << n) | (x >>> (32 - n))) & 0xFFFFFFFF
}
function ts(x, y) {
  return ((0xFFFF & x) << 16 | (0xFFFF & y)) >>> 0
}

function sm3(msgBytes) {
  const len = msgBytes.length
  // 填充
  const bitLen = len * 8
  const padLen = ((len + 1 + 8) % 64 === 0) ? 0 : (64 - ((len + 1 + 8) % 64))
  const padded = new Uint8Array(len + 1 + padLen + 8)
  padded.set(msgBytes)
  padded[len] = 0x80
  // 写入长度（大端 64 位）
  const hi = Math.floor(bitLen / 0x100000000)
  const lo = bitLen >>> 0
  padded[len + 1 + padLen] = (hi >>> 24) & 0xff
  padded[len + 1 + padLen + 1] = (hi >>> 16) & 0xff
  padded[len + 1 + padLen + 2] = (hi >>> 8) & 0xff
  padded[len + 1 + padLen + 3] = hi & 0xff
  padded[len + 1 + padLen + 4] = (lo >>> 24) & 0xff
  padded[len + 1 + padLen + 5] = (lo >>> 16) & 0xff
  padded[len + 1 + padLen + 6] = (lo >>> 8) & 0xff
  padded[len + 1 + padLen + 7] = lo & 0xff

  let V = [...IV]

  for (let i = 0; i < padded.length; i += 64) {
    const W = new Array(68)
    for (let j = 0; j < 16; j++) {
      W[j] = ((padded[i + j * 4] << 24) | (padded[i + j * 4 + 1] << 16) | (padded[i + j * 4 + 2] << 8) | padded[i + j * 4 + 3]) >>> 0
    }
    for (let j = 16; j < 68; j++) {
      W[j] = (W[j - 16] ^ W[j - 9] ^ rotl(W[j - 3], 15)) >>> 0
      W[j] = (W[j] ^ rotl(W[j - 13], 7) ^ W[j - 6]) >>> 0
    }
    const W1 = new Array(64)
    for (let j = 0; j < 64; j++) {
      W1[j] = (W[j] ^ W[j + 4]) >>> 0
    }

    let A = V[0], B = V[1], C = V[2], D = V[3]
    let E = V[4], F = V[5], G = V[6], H = V[7]

    for (let j = 0; j < 64; j++) {
      const Tj = j < 16 ? 0x79cc4519 : 0x7a879d8a
      const SS1 = rotl((rotl(A, 12) + E + rotl(Tj, j % 32)) >>> 0, 7)
      const SS2 = (SS1 ^ rotl(A, 12)) >>> 0
      const FF = j < 16 ? (A ^ B ^ C) : ((A & B) | (A & C) | (B & C))
      const GG = j < 16 ? (E ^ F ^ G) : ((E & F) | ((~E) & G))
      const TT1 = (FF + D + SS2 + W1[j]) >>> 0
      const TT2 = (GG + H + SS1 + W[j]) >>> 0
      D = C
      C = rotl(B, 9)
      B = A
      A = TT1
      H = G
      G = rotl(F, 19)
      F = E
      E = TT2
    }

    V = [
      (V[0] ^ A) >>> 0, (V[1] ^ B) >>> 0, (V[2] ^ C) >>> 0, (V[3] ^ D) >>> 0,
      (V[4] ^ E) >>> 0, (V[5] ^ F) >>> 0, (V[6] ^ G) >>> 0, (V[7] ^ H) >>> 0
    ]
  }

  return V.map(v => ('00000000' + v.toString(16)).slice(-8)).join('')
}

const hash = computed(() => {
  if (!input.value) return ''
  try {
    const bytes = new TextEncoder().encode(input.value)
    return sm3(bytes)
  } catch {
    return 'Error'
  }
})

function copy() {
  navigator.clipboard?.writeText(hash.value)
}
</script>

<style scoped>
.sm3-hash { display: flex; flex-direction: column; gap: 12px; }
.sm3-hash textarea { width: 100%; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.sm3-hash__result { display: flex; flex-direction: column; gap: 4px; }
.sm3-hash__result label { font-size: 12px; color: var(--text-tertiary); }
.sm3-hash__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.sm3-hash__result button { align-self: flex-start; margin-top: 4px; padding: 6px 16px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 13px; cursor: pointer; }
.sm3-hash__info { padding: 8px 12px; border-radius: 6px; background: var(--bg-base); font-size: 12px; color: var(--text-tertiary); }
</style>
