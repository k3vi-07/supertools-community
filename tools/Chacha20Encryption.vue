<template>
  <h-single-layout>
    <div class="chacha">
      <div class="chacha__field">
        <label>密钥 (Hex, 32 字节 = 64 字符)</label>
        <input v-model="keyHex" spellcheck="false" />
        <button class="chacha__rand" @click="randomKey">随机生成</button>
      </div>
      <div class="chacha__field">
        <label>Nonce (Hex, 12 字节 = 24 字符)</label>
        <input v-model="nonceHex" spellcheck="false" />
        <button class="chacha__rand" @click="randomNonce">随机生成</button>
      </div>
      <textarea v-model="input" rows="3" placeholder="输入明文..." spellcheck="false"></textarea>
      <div class="chacha__actions">
        <button class="chacha__btn chacha__btn--primary" @click="doEncrypt">加密</button>
        <button class="chacha__btn" @click="doDecrypt">解密</button>
      </div>
      <div class="chacha__result">
        <label>结果 (Hex)</label>
        <div class="chacha__output selectable">{{ output || '(点击加密/解密)' }}</div>
        <button v-if="output" @click="copyOutput">复制</button>
      </div>
      <div v-if="error" class="chacha__error">{{ error }}</div>
      <div class="chacha__info">
        ChaCha20 是一种高速流密码，由 Daniel J. Bernstein 设计。TLS 1.3 和 WireGuard 均采用此算法
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const keyHex = ref('000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f')
const nonceHex = ref('000000090000004a00000000')
const input = ref('Hello ChaCha20!')
const output = ref('')
const error = ref('')

function randomKey() {
  const arr = new Uint8Array(32)
  crypto.getRandomValues(arr)
  keyHex.value = Array.from(arr).map(b => b.toString(16).padStart(2, '0')).join('')
}
function randomNonce() {
  const arr = new Uint8Array(12)
  crypto.getRandomValues(arr)
  nonceHex.value = Array.from(arr).map(b => b.toString(16).padStart(2, '0')).join('')
}

function hexToBytes(hex) {
  const bytes = []
  for (let i = 0; i < hex.length; i += 2) bytes.push(parseInt(hex.substr(i, 2), 16))
  return bytes
}
function bytesToHex(bytes) {
  return bytes.map(b => b.toString(16).padStart(2, '0')).join('')
}
function strToBytes(str) {
  return Array.from(new TextEncoder().encode(str))
}
function bytesToStr(bytes) {
  return new TextDecoder().decode(new Uint8Array(bytes))
}

function rotl32(x, n) {
  return ((x << n) | (x >>> (32 - n))) >>> 0
}
function u32(a, b, c, d) {
  return [a >>> 0, b >>> 0, c >>> 0, d >>> 0]
}

function qr(state, a, b, c, d) {
  state[a] = (state[a] + state[b]) >>> 0; state[d] = rotl32(state[d] ^ state[a], 16)
  state[c] = (state[c] + state[d]) >>> 0; state[b] = rotl32(state[b] ^ state[c], 12)
  state[a] = (state[a] + state[b]) >>> 0; state[d] = rotl32(state[d] ^ state[a], 8)
  state[c] = (state[c] + state[d]) >>> 0; state[b] = rotl32(state[b] ^ state[c], 7)
}

function chacha20Block(key, counter, nonce) {
  const constants = [0x61707865, 0x3320646e, 0x79622d32, 0x6b206574]
  const state = [
    ...constants,
    ...key,
    counter & 0xFFFFFFFF,
    ...nonce
  ]
  const working = [...state]
  for (let i = 0; i < 10; i++) {
    qr(working, 0, 4, 8, 12)
    qr(working, 1, 5, 9, 13)
    qr(working, 2, 6, 10, 14)
    qr(working, 3, 7, 11, 15)
    qr(working, 0, 5, 10, 15)
    qr(working, 1, 6, 11, 12)
    qr(working, 2, 7, 8, 13)
    qr(working, 3, 4, 9, 14)
  }
  return state.map((s, i) => (s + working[i]) >>> 0)
}

function u32ToBytes(w) {
  const bytes = []
  for (let i = 0; i < w.length; i++) {
    bytes.push(w[i] & 0xff, (w[i] >>> 8) & 0xff, (w[i] >>> 16) & 0xff, (w[i] >>> 24) & 0xff)
  }
  return bytes
}
function bytesToU32(bytes) {
  const words = []
  for (let i = 0; i < bytes.length; i += 4) {
    words.push((bytes[i] | (bytes[i+1] << 8) | (bytes[i+2] << 16) | (bytes[i+3] << 24)) >>> 0)
  }
  return words
}

function chacha20Crypt(keyBytes, nonceBytes, dataBytes) {
  // Convert key to 8 u32 words
  const key = bytesToU32(keyBytes)
  const nonce = bytesToU32(nonceBytes)
  const result = []
  let counter = 1

  for (let i = 0; i < dataBytes.length; i += 64) {
    const block = chacha20Block(key, counter, nonce)
    const keystream = u32ToBytes(block)
    const chunk = dataBytes.slice(i, i + 64)
    for (let j = 0; j < chunk.length; j++) {
      result.push(chunk[j] ^ keystream[j])
    }
    counter++
  }
  return result
}

function doEncrypt() {
  error.value = ''
  try {
    if (keyHex.value.length !== 64) throw new Error('密钥必须为 64 个 hex 字符 (32 字节)')
    if (nonceHex.value.length !== 24) throw new Error('Nonce 必须为 24 个 hex 字符 (12 字节)')
    const keyBytes = hexToBytes(keyHex.value)
    const nonceBytes = hexToBytes(nonceHex.value)
    const dataBytes = strToBytes(input.value)
    const cipher = chacha20Crypt(keyBytes, nonceBytes, dataBytes)
    output.value = bytesToHex(cipher)
  } catch (e) {
    error.value = e.message
    output.value = ''
  }
}

function doDecrypt() {
  error.value = ''
  try {
    if (keyHex.value.length !== 64) throw new Error('密钥必须为 64 个 hex 字符 (32 字节)')
    if (nonceHex.value.length !== 24) throw new Error('Nonce 必须为 24 个 hex 字符 (12 字节)')
    const keyBytes = hexToBytes(keyHex.value)
    const nonceBytes = hexToBytes(nonceHex.value)
    const dataBytes = hexToBytes(input.value.trim())
    const plain = chacha20Crypt(keyBytes, nonceBytes, dataBytes)
    output.value = bytesToStr(plain)
  } catch (e) {
    error.value = e.message
    output.value = ''
  }
}

function copyOutput() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.chacha { display: flex; flex-direction: column; gap: 12px; }
.chacha__field { display: flex; flex-direction: column; gap: 4px; position: relative; }
.chacha__field label { font-size: 12px; color: var(--text-tertiary); }
.chacha__field input { padding: 8px 10px; padding-right: 70px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; }
.chacha__rand { position: absolute; right: 6px; bottom: 6px; padding: 4px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-secondary); font-size: 11px; cursor: pointer; }
.chacha textarea { width: 100%; padding: 10px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.chacha__actions { display: flex; gap: 8px; }
.chacha__btn { padding: 8px 20px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; cursor: pointer; }
.chacha__btn--primary { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.chacha__result { display: flex; flex-direction: column; gap: 4px; }
.chacha__result label { font-size: 12px; color: var(--text-tertiary); }
.chacha__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; min-height: 40px; }
.chacha__result button { align-self: flex-start; padding: 4px 12px; border: none; border-radius: 4px; background: var(--color-primary); color: white; font-size: 12px; cursor: pointer; }
.chacha__error { padding: 8px 12px; border-radius: 6px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
.chacha__info { padding: 8px 12px; border-radius: 6px; background: var(--bg-base); font-size: 12px; color: var(--text-tertiary); }
</style>
