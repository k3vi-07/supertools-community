<template>
  <h-single-layout>
    <div class="sm4-tool">
      <div class="sm4-tool__field"><label>密钥 (16字节 Hex，32 字符)</label><input v-model="keyHex" placeholder="0123456789abcdef0123456789abcdef" /></div>
      <div class="sm4-tool__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文':'Hex密文'" spellcheck="false"></textarea></div>
      <div class="sm4-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">SM4 加密 (ECB)</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">SM4 解密</button>
      </div>
      <div class="sm4-tool__output selectable">{{ output }}</div>
      <div class="sm4-tool__hint">SM4 国密分组密码算法，密钥和数据均为 16 字节对齐</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('0123456789abcdef0123456789abcdef')
const input = ref('0123456789abcdeffedcba9876543210')
const mode = ref('encrypt')

// SM4 S-Box
const SBOX = [
  0xd6,0x90,0xe9,0xfe,0xcc,0xe1,0x3d,0xb7,0x16,0xb6,0x14,0xc2,0x28,0xfb,0x2c,0x05,
  0x2b,0x67,0x9a,0x76,0x2a,0xbe,0x04,0xc3,0xaa,0x44,0x13,0x26,0x49,0x86,0x06,0x99,
  0x9c,0x42,0x50,0xf4,0x91,0xef,0x98,0x7a,0x33,0x54,0x0b,0x43,0xed,0xcf,0xac,0x62,
  0xe4,0xb3,0x1c,0xa9,0xc9,0x08,0xe8,0x95,0x80,0xdf,0x94,0xfa,0x75,0x8f,0x3f,0xa6,
  0x47,0x07,0xa7,0xfc,0xf3,0x73,0x17,0xba,0x83,0x59,0x3c,0x19,0xe6,0x85,0x4f,0xa8,
  0x68,0x6b,0x81,0xb2,0x71,0x64,0xda,0x8b,0xf8,0xeb,0x0f,0x4b,0x70,0x56,0x9d,0x35,
  0x1e,0x24,0x0e,0x5e,0x63,0x58,0xd1,0xa2,0x25,0x22,0x7c,0x3b,0x01,0x21,0x78,0x87,
  0xd4,0x00,0x46,0x57,0x9f,0xd3,0x27,0x52,0x4c,0x36,0x02,0xe7,0xa0,0xc4,0xc8,0x9e,
  0xea,0xbf,0x8a,0xd2,0x40,0xc7,0x38,0xb5,0xa3,0xf7,0xf2,0xce,0xf9,0x61,0x15,0xa1,
  0xe0,0xae,0x5d,0xa4,0x9b,0x34,0x1a,0x55,0xad,0x93,0x32,0x30,0xf5,0x8c,0xb1,0xe3,
  0x1d,0xf6,0xe2,0x2e,0x82,0x66,0xca,0x60,0xc0,0x29,0x23,0xab,0x0d,0x53,0x4e,0x6f,
  0xd5,0xdb,0x37,0x45,0xde,0xfd,0x8e,0x2f,0x03,0xff,0x6a,0x72,0x6d,0x6c,0x5b,0x51,
  0x8d,0x1b,0xaf,0x92,0xbb,0xdd,0xbc,0x7f,0x11,0xd9,0x5c,0x41,0x1f,0x10,0x5a,0xd8,
  0x0a,0xc1,0x31,0x88,0xa5,0xcd,0x7b,0xbd,0x2d,0x74,0xd0,0x12,0xb8,0xe5,0xb4,0xb0,
  0x89,0x69,0x97,0x4a,0x0c,0x96,0x77,0x7e,0x65,0x b9,0xf1,0x09,0xc5,0x6e,0xc6,0x84,
  0x18,0xf0,0x7d,0xec,0x3a,0xdc,0x4d,0x20,0x79,0xee,0x5f,0x3e,0xd7,0xcb,0x39,0x48
]
// Fix the typo in SBOX (0x b9 -> 0xb9)
SBOX[0xe9] = 0xb9

const FK = [0xa3b1bac6, 0x56aa3350, 0x677d9197, 0xb27022dc]

function rotl(x, n) { return ((x << n) | (x >>> (32 - n))) >>> 0 }

const tau = (a) => ((SBOX[(a >>> 24) & 0xFF] << 24) | (SBOX[(a >>> 16) & 0xFF] << 16) | (SBOX[(a >>> 8) & 0xFF] << 8) | SBOX[a & 0xFF]) >>> 0
const T = (x) => { const t = tau(x); return (t ^ rotl(t, 2) ^ rotl(t, 10) ^ rotl(t, 18) ^ rotl(t, 24)) >>> 0 }
const Tprime = (x) => { const t = tau(x); return (t ^ rotl(t, 13) ^ rotl(t, 23)) >>> 0 }

function keyExpansion(key) {
  const K = [FK[0] ^ key[0], FK[1] ^ key[1], FK[2] ^ key[2], FK[3] ^ key[3]]
  const rk = []
  for (let i = 0; i < 32; i++) {
    const k = (K[i] ^ Tprime(K[i+1] ^ K[i+2] ^ K[i+3] ^ CK[i])) >>> 0
    K.push(k); rk.push(k)
  }
  return rk
}

const CK = Array.from({length: 32}, (_, i) => {
  const b = [(4*i+0)*7%256, (4*i+1)*7%256, (4*i+2)*7%256, (4*i+3)*7%256]
  return ((b[0]<<24)|(b[1]<<16)|(b[2]<<8)|b[3])>>>0
})

function cryptBlock(input, rk) {
  const X = [...input]
  for (let i = 0; i < 32; i++) {
    const x = (X[i] ^ T(X[i+1] ^ X[i+2] ^ X[i+3] ^ rk[i])) >>> 0
    X.push(x)
  }
  return [X[35], X[34], X[33], X[32]]
}

function hexToWords(hex) {
  const words = []
  for (let i = 0; i < hex.length; i += 8) words.push(parseInt(hex.substr(i,8), 16) >>> 0)
  return words
}
function wordsToHex(words) { return words.map(w => (w >>> 0).toString(16).padStart(8,'0')).join('') }

const output = computed(() => {
  try {
    const key = hexToWords(keyHex.value.replace(/\s/g,''))
    if (key.length !== 4) return '❌ 密钥必须 32 Hex 字符'
    const rk = keyExpansion(key)
    if (mode.value === 'encrypt') {
      const data = hexToWords(input.value.replace(/\s/g,''))
      if (data.length % 4 !== 0) return '❌ 数据必须 16 字节对齐'
      let result = []
      for (let i = 0; i < data.length; i += 4) result.push(...cryptBlock(data.slice(i, i+4), rk))
      return wordsToHex(result)
    } else {
      const rkRev = [...rk].reverse()
      const data = hexToWords(input.value.replace(/\s/g,''))
      let result = []
      for (let i = 0; i < data.length; i += 4) result.push(...cryptBlock(data.slice(i, i+4), rkRev))
      return wordsToHex(result)
    }
  } catch(e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.sm4-tool { display: flex; flex-direction: column; gap: 12px; }
.sm4-tool__field { display: flex; flex-direction: column; gap: 4px; }
.sm4-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.sm4-tool__field input, .sm4-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.sm4-tool__actions { display: flex; gap: 8px; }
.sm4-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.sm4-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.sm4-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.sm4-tool__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
