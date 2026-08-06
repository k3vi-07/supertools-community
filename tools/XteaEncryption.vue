<template>
  <h-single-layout>
    <div class="xtea-tool">
      <div class="xtea-tool__field"><label>密钥 (128位/16字节文本)</label><input v-model="key" placeholder="16字符密钥" /></div>
      <div class="xtea-tool__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文':'Hex密文'" spellcheck="false"></textarea></div>
      <div class="xtea-tool__type">
        <button :class="{active:algo==='xtea'}" @click="algo='xtea'">XTEA</button>
        <button :class="{active:algo==='xxtea'}" @click="algo='xxtea'">XXTEA</button>
      </div>
      <div class="xtea-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">加密</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button>
      </div>
      <div class="xtea-tool__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const key = ref('1234567890abcdef')
const input = ref('Hello XTEA!')
const mode = ref('encrypt')
const algo = ref('xtea')

const DELTA = 0x9E3779B9

function strToUint32(str) {
  const bytes = new TextEncoder().encode(str)
  const padded = [...bytes]
  while (padded.length % 4 !== 0) padded.push(0)
  const result = []
  for (let i = 0; i < padded.length; i += 4) {
    result.push((padded[i] << 24) | (padded[i+1] << 16) | (padded[i+2] << 8) | padded[i+3])
  }
  return { words: result, originalLen: bytes.length }
}

function uint32ToStr(words, len) {
  const bytes = []
  for (const w of words) {
    bytes.push((w >>> 24) & 0xFF, (w >>> 16) & 0xFF, (w >>> 8) & 0xFF, w & 0xFF)
  }
  return new TextDecoder().decode(new Uint8Array(bytes.slice(0, len || bytes.length)))
}

function getKeyWords(keyStr) {
  const padded = (keyStr + '\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0').slice(0, 16)
  const result = []
  const bytes = new TextEncoder().encode(padded)
  for (let i = 0; i < 16; i += 4) {
    result.push(((bytes[i] || 0) << 24) | ((bytes[i+1] || 0) << 16) | ((bytes[i+2] || 0) << 8) | (bytes[i+3] || 0))
  }
  return result
}

// XTEA encrypt one 64-bit block (v0, v1)
function xteaEncryptBlock(v, k) {
  let [v0, v1] = v
  let sum = 0
  for (let i = 0; i < 32; i++) {
    v0 = (v0 + ((((v1 << 4) ^ (v1 >>> 5)) + v1) ^ (sum + k[sum & 3]))) | 0
    sum = (sum + DELTA) | 0
    v1 = (v1 + ((((v0 << 4) ^ (v0 >>> 5)) + v0) ^ (sum + k[(sum >>> 11) & 3]))) | 0
  }
  return [v0, v1]
}

function xteaDecryptBlock(v, k) {
  let [v0, v1] = v
  let sum = (DELTA * 32) | 0
  for (let i = 0; i < 32; i++) {
    v1 = (v1 - ((((v0 << 4) ^ (v0 >>> 5)) + v0) ^ (sum + k[(sum >>> 11) & 3]))) | 0
    sum = (sum - DELTA) | 0
    v0 = (v0 - ((((v1 << 4) ^ (v1 >>> 5)) + v1) ^ (sum + k[sum & 3]))) | 0
  }
  return [v0, v1]
}

// XXTEA encrypt/decrypt array of words
function xxteaEncrypt(v, k) {
  const n = v.length
  if (n < 2) return v
  let z = v[n - 1], y = v[0], sum = 0
  const rounds = 6 + 52 / n
  for (let i = 0; i < rounds; i++) {
    sum = (sum + DELTA) | 0
    const e = (sum >>> 2) & 3
    for (let p = 0; p < n - 1; p++) {
      y = v[p + 1]
      z = v[p] = (v[p] + ((((z >>> 5) ^ (y << 2)) + ((y >>> 3) ^ (z << 4))) ^ ((sum ^ y) + (k[(p & 3) ^ e] ^ z)))) | 0
    }
    y = v[0]
    z = v[n - 1] = (v[n - 1] + ((((z >>> 5) ^ (y << 2)) + ((y >>> 3) ^ (z << 4))) ^ ((sum ^ y) + (k[(n - 1) & 3 ^ e] ^ z)))) | 0
  }
  return v
}

const output = computed(() => {
  if (!input.value) return ''
  try {
    const k = getKeyWords(key.value)
    if (algo.value === 'xtea') {
      const { words } = strToUint32(input.value)
      if (mode.value === 'encrypt') {
        const result = []
        for (let i = 0; i < words.length; i += 2) result.push(...xteaEncryptBlock([words[i] || 0, words[i+1] || 0], k))
        return result.map(w => (w >>> 0).toString(16).padStart(8, '0')).join('')
      } else {
        const hex = input.value.replace(/\s/g, '')
        const words = []
        for (let i = 0; i < hex.length; i += 8) words.push(parseInt(hex.substr(i, 8), 16) | 0)
        const result = []
        for (let i = 0; i < words.length; i += 2) result.push(...xteaDecryptBlock([words[i], words[i+1]], k))
        return uint32ToStr(result)
      }
    } else {
      if (mode.value === 'encrypt') {
        const { words } = strToUint32(input.value)
        const enc = xxteaEncrypt([...words], k)
        return enc.map(w => (w >>> 0).toString(16).padStart(8, '0')).join('')
      } else {
        const hex = input.value.replace(/\s/g, '')
        const words = []
        for (let i = 0; i < hex.length; i += 8) words.push(parseInt(hex.substr(i, 8), 16) | 0)
        return uint32ToStr(xxteaDecrypt([...words], k))
      }
    }
  } catch (e) { return '❌ ' + e.message }
})

function xxteaDecrypt(v, k) {
  const n = v.length
  if (n < 2) return v
  const rounds = 6 + 52 / n
  let sum = (DELTA * rounds) | 0
  let z, y = v[0]
  for (let i = 0; i < rounds; i++) {
    const e = (sum >>> 2) & 3
    for (let p = n - 1; p > 0; p--) {
      z = v[p - 1]
      y = v[p] = (v[p] - ((((z >>> 5) ^ (y << 2)) + ((y >>> 3) ^ (z << 4))) ^ ((sum ^ y) + (k[(p & 3) ^ e] ^ z)))) | 0
    }
    z = v[n - 1]
    y = v[0] = (v[0] - ((((z >>> 5) ^ (y << 2)) + ((y >>> 3) ^ (z << 4))) ^ ((sum ^ y) + (k[e] ^ z)))) | 0
    sum = (sum - DELTA) | 0
  }
  return v
}
</script>
<style scoped>
.xtea-tool { display: flex; flex-direction: column; gap: 12px; }
.xtea-tool__field { display: flex; flex-direction: column; gap: 4px; }
.xtea-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.xtea-tool__field input, .xtea-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.xtea-tool__type, .xtea-tool__actions { display: flex; gap: 8px; }
.xtea-tool__type button, .xtea-tool__actions button { padding: 6px 14px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.xtea-tool__type button.active, .xtea-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.xtea-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
</style>
