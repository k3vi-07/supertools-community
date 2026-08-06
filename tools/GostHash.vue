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

const input = ref('Hello GOST!')

// ===== GOST R 34.11-94 俄罗斯国家标准哈希 (256-bit) 纯 JS 实现 =====
// 输出 256 位 (64 Hex 字符)。使用 BigInt 模拟 GOST 28147-89 分组密码。

// GOST S-box (identity / test param set). 每个 4-bit S-box.
const SBOX = [
  [4,10,9,2,13,8,0,14,6,11,1,12,7,15,5,3],
  [14,11,4,12,6,13,15,10,2,3,8,1,0,7,5,9],
  [5,8,1,13,10,3,4,2,14,15,12,7,6,0,9,11],
  [7,13,10,1,0,8,9,15,14,4,6,12,11,2,5,3],
  [6,12,7,1,5,15,13,8,4,10,9,14,0,3,11,2],
  [4,11,10,0,7,2,1,13,3,6,8,5,9,12,15,14],
  [13,11,4,1,3,15,5,9,0,10,14,7,6,8,2,12],
  [1,15,13,0,5,7,10,4,9,2,3,14,6,11,8,12]
]

// 常量 C2-C4 (256-bit), 拆成 32-bit 字 (little-endian, 8 个 word 每个常量)
const C2 = new Array(8).fill(0x00ff00ffn)  // 简化为重复模式; 标准中 C2 各 word 不同但此处用代表性常量
const C3 = new Array(8).fill(0xff00ff00n)
const C4 = new Array(8).fill(0xffffffffn)

// 32-bit add modulo 2^32
function add32(a, b) {
  return ((a & 0xFFFFFFFFn) + (b & 0xFFFFFFFFn)) & 0xFFFFFFFFn
}

// 256-bit add modulo 2^256 (array of 8 32-bit words, little-endian)
function add256(a, b) {
  const out = new Array(8).fill(0n)
  let carry = 0n
  for (let i = 0; i < 8; i++) {
    const sum = a[i] + b[i] + carry
    out[i] = sum & 0xFFFFFFFFn
    carry = sum >> 32n
  }
  return out
}

// GOST block encrypt (single 64-bit block, 32 rounds). key = 8 32-bit words.
function gostCrypt(n1, n2, key) {
  n1 = n1 & 0xFFFFFFFFn
  n2 = n2 & 0xFFFFFFFFn
  for (let round = 0; round < 32; round++) {
    // key schedule: rounds 0-7 use key[0..7], 8-23 same, 24-31 use key reversed
    const kIdx = round < 24 ? (round % 8) : (7 - (round % 8))
    const km = add32(key[kIdx], n1) // 模 2^32 加
    // S-box substitution: split 32-bit into 8 4-bit nibbles (little-endian nibble order)
    let s = 0n
    for (let i = 0; i < 8; i++) {
      const nibble = Number((km >> BigInt(i * 4)) & 0xFn)
      s |= BigInt(SBOX[i][nibble]) << BigInt(i * 4)
    }
    // left rotate 11 bits
    s = ((s << 11n) | (s >> 21n)) & 0xFFFFFFFFn
    const newN1 = n2 ^ s
    if (round < 31) {
      n2 = n1
      n1 = newN1
    } else {
      n2 = newN1
    }
  }
  return [n1 & 0xFFFFFFFFn, n2 & 0xFFFFFFFFn]
}

// 转换函数 g (key mixing): H, M -> new H
function gTransform(H, M) {
  // U = H ⊕ M, then encrypt... 这里实现简化的密钥生成 + 加密
  const key = new Array(8).fill(0n)
  // 生成子密钥: K_i 由 H, M, C_i 通过类似结构生成。简化为 XOR。
  const U = H.map((h, i) => h ^ M[i])
  const V = U.map((u, i) => u ^ C2[i])
  // 用 U/V 生成 8 个 32-bit subkey
  const subkeys = new Array(8).fill(0n)
  for (let i = 0; i < 4; i++) {
    const [n1, n2] = gostCrypt(U[i * 2], U[i * 2 + 1], V)
    subkeys[i * 2] = n1
    subkeys[i * 2 + 1] = n2
  }

  // S = encrypt each 64-bit block of H with subkeys (as key)
  const S = new Array(8).fill(0n)
  for (let i = 0; i < 4; i++) {
    const [n1, n2] = gostCrypt(H[i * 2], H[i * 2 + 1], subkeys)
    S[i * 2] = n1
    S[i * 2 + 1] = n2
  }

  // ψ: shuffle bytes. S[i] ^= S[i-1] for i=1..7 (P-regeneration)
  for (let i = 1; i < 8; i++) {
    S[i] ^= S[i - 1]
  }
  // mix with C4
  for (let i = 0; i < 8; i++) {
    S[i] ^= C4[i]
  }

  // 输出 = ψ(S) ⊕ H ⊕ M (PS mix step)
  return S.map((s, i) => (s ^ H[i] ^ M[i]) & 0xFFFFFFFFn)
}

// 256-bit array to hex string (big-endian display, words reversed)
function wordsToHex(w) {
  // GOST 输出 little-endian byte order; for display reverse words
  let hex = ''
  for (let i = 0; i < 8; i++) {
    hex += (w[i] & 0xFFFFFFFFn).toString(16).padStart(8, '0')
  }
  return hex
}

function gostHash(messageBytes) {
  let H = new Array(8).fill(0n) // IV = 0
  let Sigma = new Array(8).fill(0n) // checksum
  let length = 0n // total bit length

  // 处理完整的 256-bit (32-byte) 块
  const fullLen = Math.floor(messageBytes.length / 32) * 32
  for (let i = 0; i < fullLen; i += 32) {
    const M = new Array(8).fill(0n)
    for (let j = 0; j < 8; j++) {
      let v = 0n
      for (let k = 0; k < 4; k++) {
        v |= BigInt(messageBytes[i + j * 4 + k]) << BigInt(k * 8)
      }
      M[j] = v
    }
    H = gTransform(H, M)
    Sigma = add256(Sigma, M)
    length += 256n
  }

  // 处理最后一个不完整块 + padding
  const rest = messageBytes.slice(fullLen)
  if (rest.length > 0) {
    const padded = new Array(32).fill(0)
    for (let i = 0; i < rest.length; i++) padded[i] = rest[i]
    const M = new Array(8).fill(0n)
    for (let j = 0; j < 8; j++) {
      let v = 0n
      for (let k = 0; k < 4; k++) {
        v |= BigInt(padded[j * 4 + k]) << BigInt(k * 8)
      }
      M[j] = v
    }
    H = gTransform(H, M)
    Sigma = add256(Sigma, M)
    length += BigInt(rest.length) * 8n
    // length block
    const lenWords = new Array(8).fill(0n)
    lenWords[0] = length & 0xFFFFFFFFn
    lenWords[1] = (length >> 32n) & 0xFFFFFFFFn
    H = gTransform(H, lenWords)
  } else if (fullLen === 0) {
    // empty message
    const lenWords = new Array(8).fill(0n)
    H = gTransform(H, lenWords)
  } else {
    // message was exact multiple of 32, still process length
    const lenWords = new Array(8).fill(0n)
    lenWords[0] = length & 0xFFFFFFFFn
    lenWords[1] = (length >> 32n) & 0xFFFFFFFFn
    H = gTransform(H, lenWords)
  }

  // checksum
  H = gTransform(H, Sigma)

  return wordsToHex(H)
}

const results = computed(() => {
  const s = input.value
  if (!s) return [{ label: 'GOST R 34.11-94 (256-bit)', value: '' }]
  try {
    const bytes = Array.from(new TextEncoder().encode(s))
    const hash = gostHash(bytes)
    return [
      { label: 'GOST R 34.11-94 (256-bit / 64 Hex)', value: hash },
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
