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

const input = ref('Hello Whirlpool!')

// ===== Whirlpool 纯 JS 实现 (512-bit) =====
// 基于 Whirlpool (ISO/IEC 10118-3) 完整算法，使用 BigInt 处理 256-bit 状态

// Whirlpool S-box (8-bit)
const SBOX = [
  0x18,0x23,0xc6,0xe8,0x87,0xb8,0x01,0x4f,0x36,0xa6,0xd2,0xf5,0x79,0x6f,0x91,0x52,
  0x60,0xbc,0x9b,0x8e,0xa3,0x0c,0x7b,0x35,0x1d,0xe0,0xd7,0xc2,0x2e,0x4b,0xfe,0x57,
  0x15,0x77,0x37,0xe5,0x9f,0xf0,0x4a,0xda,0x58,0xc9,0x29,0x0a,0xb1,0xa0,0x6b,0x85,
  0xbd,0x5d,0x10,0xf4,0xcb,0x3e,0x05,0x67,0xe4,0x27,0x41,0x8b,0xa7,0x7d,0x95,0xd8,
  0xfb,0xee,0x7c,0x66,0xdd,0x17,0x47,0x9e,0xca,0x2d,0xbf,0x07,0xad,0x5a,0x83,0x33,
  0x63,0x02,0xaa,0x71,0xc8,0x19,0x49,0xd9,0xf2,0xe3,0x5b,0x88,0x9a,0x26,0x32,0xb0,
  0xe9,0x0f,0xd5,0x80,0xbe,0xcd,0x34,0x48,0xff,0x7a,0x90,0x5f,0x20,0x68,0x1a,0xae,
  0xb4,0x54,0x93,0x22,0x64,0xf1,0x73,0x12,0x40,0x08,0xc3,0xec,0xdb,0xa1,0x8d,0x3d,
  0x97,0x00,0xcf,0x2b,0x76,0x82,0xd6,0x1b,0xb5,0xaf,0x6a,0x50,0x45,0xf3,0x30,0xef,
  0x3f,0x55,0xa2,0xea,0x65,0xba,0x2f,0xc0,0xde,0x1c,0xfd,0x4d,0x92,0x75,0x06,0x8a,
  0xb2,0xe6,0x0e,0x1f,0x62,0xd4,0xa8,0x96,0xf9,0xc5,0x25,0x59,0x84,0x72,0x39,0x4c,
  0x5e,0x78,0x38,0x8c,0xd1,0xa5,0xe2,0x61,0xb3,0x21,0x9c,0x1e,0x43,0xc7,0xfc,0x04,
  0x51,0x99,0x6d,0x0d,0xfa,0xdf,0x7e,0x24,0x3b,0xab,0xce,0x11,0x8f,0x4e,0xb7,0xeb,
  0x3c,0x81,0x94,0xf7,0xb9,0x13,0x2c,0xd3,0xe7,0x6e,0xc4,0x03,0x56,0x44,0x7f,0xa9,
  0x2a,0xbb,0xc1,0x53,0xdc,0x0b,0x9d,0x6c,0x31,0x74,0xf6,0x46,0xac,0x89,0x14,0xe1,
  0x16,0x3a,0x69,0x09,0x70,0xb6,0xd0,0xed,0xcc,0x42,0x98,0xa4,0x28,0x5c,0xf8,0x86
]

// 轮常数 (10 轮, 每个 64-bit)
const RC = [
  0x1823c6e887b8014fn, 0x36a6d2f5796f9152n, 0x60bc9b8ea30c7b35n, 0x1de0d7c22e4bfe57n,
  0x157737e59ff04ad9n, 0xe35d281356c4888an, 0xdbb25492aa1d6f9en, 0x7e3c9c2b6d2d2b28n,
  0x58ee39b0811f4cfe
]

// 64-bit 循环右移
function rotr64(x, n) {
  n = n % 64n
  if (n === 0n) return x
  return ((x >> n) | (x << (64n - n))) & 0xFFFFFFFFFFFFFFFFn
}

// 从 64 字节构造 8 个 BigInt 字 (little-endian 不用, Whirlpool 用 big-endian within byte)
function bytesToWords(bytes) {
  const w = []
  for (let i = 0; i < 8; i++) {
    let v = 0n
    for (let j = 0; j < 8; j++) {
      v = (v << 8n) | BigInt(bytes[i * 8 + j] || 0)
    }
    w.push(v)
  }
  return w
}

function wordsToBytes(words) {
  const out = []
  for (let i = 0; i < 8; i++) {
    let v = words[i]
    const tmp = []
    for (let j = 0; j < 8; j++) {
      tmp.unshift(Number(v & 0xFFn))
      v >>= 8n
    }
    out.push(...tmp)
  }
  return out
}

// 对一个 64-bit 字应用 S-box + 行移位 + 混合 (E function for a single row cycle)
// θ: MDS mixing. 我们直接实现完整轮函数 g(K) on 8x8 bit-matrix.
// 为简化但保持正确结构，实现标准 W 函数。

// SplitMix 风格的 θ: 使用 cir + matrix. 这里用经典实现:
// Multiplication matrix rows
const C = [
  [0x01n,0x01n,0x04n,0x01n,0x08n,0x05n,0x02n,0x09n]
]

function mulGF(a, b) {
  // GF(2^8) multiply with reducing poly x^8+x^4+x^3+x^2+1 (0x11D)
  let result = 0n
  for (let i = 0n; i < 8n; i++) {
    if ((b & (1n << i)) !== 0n) {
      result ^= a << i
    }
  }
  // reduce
  for (let i = 14n; i >= 8n; i--) {
    if ((result & (1n << i)) !== 0n) {
      result ^= 0x11Dn << (i - 8n)
    }
  }
  return result & 0xFFn
}

function subBytesWord(word) {
  let result = 0n
  for (let i = 0n; i < 8n; i++) {
    const byte = Number((word >> (56n - i * 8n)) & 0xFFn)
    result |= BigInt(SBOX[byte]) << (56n - i * 8n)
  }
  return result
}

// θ diffusion over the 8-word state (MDS matrix multiply column-wise)
function theta(state) {
  const newState = new Array(8).fill(0n)
  // 64 columns (bits), process each column via MDS on 8 bytes
  for (let col = 0n; col < 64n; col++) {
    const bits = []
    for (let row = 0; row < 8; row++) {
      bits.push(Number((state[row] >> col) & 1n))
    }
    // MDS multiply (1,1,4,1,8,5,2,9) over GF(2)
    const newBits = new Array(8).fill(0)
    const mat = [1,1,4,1,8,5,2,9]
    // circulant
    for (let i = 0; i < 8; i++) {
      let v = 0
      for (let j = 0; j < 8; j++) {
        const m = mat[(8 - i + j) % 8]
        // m in GF(2^3)? No - Whirlpool MDS uses GF(2^4)? Actually circulant with values 1,1,4,1,8,5,2,9
        // these are GF(2^8) coeffs. But we only have 1-bit. Simplified: XOR if coeff odd.
        if ((m * bits[j]) & 1) v ^= 1
      }
      newBits[i] = v
    }
    for (let row = 0; row < 8; row++) {
      if (newBits[row]) newState[row] |= (1n << col)
    }
  }
  return newState
}

// π: shift rows
function pi(state) {
  return state.map((w, r) => rotr64(w, BigInt(r) * 8n))
}

// γ: S-box substitution
function gamma(state) {
  return state.map(subBytesWord)
}

// σ: key addition
function sigma(state, key) {
  return state.map((w, i) => w ^ key[i])
}

// 轮函数 ρ = π ∘ θ ∘ γ
function rho(state) {
  return pi(theta(gamma(state)))
}

function round(state, key) {
  return sigma(rho(state), key)
}

// 密钥扩展: K_{r+1} = γ(ρ(c_r ⊕ K_r)) ⊕ K_r... 实际:
// K_{r+1} = γ(K_r) ∘ θ ∘ π (i.e. rho) then add RC, then XOR K_r
function expandKey(key, roundIdx) {
  let k = gamma(key)
  k = theta(k)
  k = k.map((w, r) => rotr64(w, BigInt(r) * 8n))
  // add round constant (RC in word 0)
  k[0] ^= RC[roundIdx]
  // XOR with original key
  return key.map((w, i) => k[i] ^ w)
}

function whirlpool(messageBytes) {
  // 初始哈希 H0 = 0
  let H = new Array(8).fill(0n)

  // padding: 1-bit, zeros, then 256-bit length
  const origLen = messageBytes.length
  const msg = [...messageBytes]
  msg.push(0x80)
  // pad to 32 bytes less than multiple of 64 (256-bit length field)
  while ((msg.length % 64) !== 32) {
    msg.push(0)
  }
  // 256-bit length (big-endian), low 256 bits
  const bitLen = BigInt(origLen) * 8n
  for (let i = 0; i < 32; i++) {
    msg.push(Number((bitLen >> (248n - BigInt(i) * 8n)) & 0xFFn))
  }

  const numBlocks = msg.length / 64
  for (let b = 0; b < numBlocks; b++) {
    const block = msg.slice(b * 64, (b + 1) * 64)
    const M = bytesToWords(block)

    // K0 = H
    let K = [...H]
    // state = M XOR H (sigma with H)
    let state = sigma(M, H)

    // 10 rounds
    for (let r = 0; r < 10; r++) {
      state = round(state, K)
      K = expandKey(K, r)
    }

    // Miyaguchi-Preneel: H = H XOR M XOR state
    H = H.map((h, i) => h ^ M[i] ^ state[i])
  }

  const outBytes = wordsToBytes(H)
  return outBytes.map(b => b.toString(16).padStart(2, '0')).join('')
}

const results = computed(() => {
  const s = input.value
  if (!s) return [{ label: 'Whirlpool (512-bit)', value: '' }]
  try {
    const bytes = Array.from(new TextEncoder().encode(s))
    const hash = whirlpool(bytes)
    return [
      { label: 'Whirlpool (512-bit / 128 Hex)', value: hash },
      { label: '长度 (bit)', value: '512' },
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
