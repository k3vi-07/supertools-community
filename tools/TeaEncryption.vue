<template>
  <h-single-layout>
    <div class="tea-tool">
      <div class="tea-tool__field"><label>密钥 (Hex, 32 字符 = 128 位)</label><input v-model="keyHex" placeholder="000102030405060708090a0b0c0d0e0f" spellcheck="false" /></div>
      <div class="tea-tool__field"><label>输入</label><textarea v-model="input" rows="3" :placeholder="mode==='encrypt'?'明文 Hex (16 字符 = 64 位)':'密文 Hex (16 字符)'" spellcheck="false"></textarea></div>
      <div class="tea-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">TEA 加密</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">TEA 解密</button>
      </div>
      <div class="tea-tool__output selectable">{{ output }}</div>
      <div class="tea-tool__hint">TEA — Tiny Encryption Algorithm。128 位密钥，64 位分组，32 轮 Feistel。Wheeler & Needham 1994</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('000102030405060708090a0b0c0d0e0f')
const input = ref('0123456789abcdef')
const mode = ref('encrypt')

// TEA magic delta constant (golden ratio derived)
const DELTA = 0x9e3779b9

// TEA encrypt: operates on two 32-bit halves (v0, v1) with four 32-bit subkeys (k[0..3]).
// 32 Feistel rounds, two rounds per loop iteration (32 / 2 = 16 iterations).
function teaEncrypt(v0, v1, k) {
  let sum = 0
  // 32 rounds total (16 iterations of 2 rounds each)
  for (let i = 0; i < 32; i++) {
    sum = (sum + DELTA) >>> 0
    // >>> 0 forces unsigned 32-bit after the left shift / add / xor
    v0 = (v0 + (((v1 << 4) + k[0]) ^ (v1 + sum) ^ ((v1 >>> 5) + k[1]))) >>> 0
    v1 = (v1 + (((v0 << 4) + k[2]) ^ (v0 + sum) ^ ((v0 >>> 5) + k[3]))) >>> 0
  }
  return [v0, v1]
}

// TEA decrypt: identical structure but sum starts at DELTA * 32 and is decremented.
function teaDecrypt(v0, v1, k) {
  let sum = (DELTA * 32) >>> 0
  for (let i = 0; i < 32; i++) {
    v1 = (v1 - (((v0 << 4) + k[2]) ^ (v0 + sum) ^ ((v0 >>> 5) + k[3]))) >>> 0
    v0 = (v0 - (((v1 << 4) + k[0]) ^ (v1 + sum) ^ ((v1 >>> 5) + k[1]))) >>> 0
    sum = (sum - DELTA) >>> 0
  }
  return [v0, v1]
}

function hexToBytes(hex) {
  const out = []
  for (let i = 0; i < hex.length; i += 2) out.push(parseInt(hex.substr(i, 2), 16))
  return out
}

const output = computed(() => {
  try {
    const kc = keyHex.value.replace(/\s/g, '').toLowerCase()
    if (kc.length !== 32) return '❌ 密钥必须为 32 Hex 字符 (128 位)'
    const keyBytes = hexToBytes(kc)
    if (keyBytes.length !== 16) return '❌ 密钥必须为 16 字节'
    // Pack key into four 32-bit big-endian words
    const k = []
    for (let i = 0; i < 4; i++) {
      k.push(((keyBytes[i*4] << 24) | (keyBytes[i*4+1] << 16) | (keyBytes[i*4+2] << 8) | keyBytes[i*4+3]) >>> 0)
    }
    const ic = input.value.replace(/\s/g, '').toLowerCase()
    if (ic.length !== 16) return '❌ 数据必须为 16 Hex 字符 (64 位)'
    const data = hexToBytes(ic)
    let v0 = ((data[0] << 24) | (data[1] << 16) | (data[2] << 8) | data[3]) >>> 0
    let v1 = ((data[4] << 24) | (data[5] << 16) | (data[6] << 8) | data[7]) >>> 0
    const [o0, o1] = mode.value === 'encrypt'
      ? teaEncrypt(v0, v1, k)
      : teaDecrypt(v0, v1, k)
    const out = [
      (o0 >>> 24) & 0xff, (o0 >>> 16) & 0xff, (o0 >>> 8) & 0xff, o0 & 0xff,
      (o1 >>> 24) & 0xff, (o1 >>> 16) & 0xff, (o1 >>> 8) & 0xff, o1 & 0xff
    ]
    return out.map(b => b.toString(16).padStart(2, '0')).join('')
  } catch (e) {
    return '❌ ' + e.message
  }
})
</script>

<style scoped>
.tea-tool { display: flex; flex-direction: column; gap: 12px; }
.tea-tool__field { display: flex; flex-direction: column; gap: 4px; }
.tea-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.tea-tool__field input, .tea-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.tea-tool__actions { display: flex; gap: 8px; }
.tea-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.tea-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.tea-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.tea-tool__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
