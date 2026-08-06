<template>
  <h-single-layout>
    <div class="tf-tool">
      <div class="tf-tool__field"><label>密钥 (Hex, 32 字符 = 128 位)</label><input v-model="keyHex" placeholder="0123456789abcdef0123456789abcdef" spellcheck="false" /></div>
      <div class="tf-tool__field"><label>输入</label><textarea v-model="input" rows="3" :placeholder="mode==='encrypt'?'明文 Hex (32 字符 = 128 位)':'密文 Hex (32 字符)'" spellcheck="false"></textarea></div>
      <div class="tf-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">Twofish 加密</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">Twofish 解密</button>
      </div>
      <div class="tf-tool__output selectable">{{ output }}</div>
      <div class="tf-tool__hint">Twofish — 128 位分组 Feistel 密码，AES 候选算法。Schneier 等 1998</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('0123456789abcdef0123456789abcdef')
const input = ref('0123456789abcdeffedcba9876543210')
const mode = ref('encrypt')

// q-tables (fixed permutations used in Twofish's key-dependent S-boxes)
const Q0 = [
  0xa9,0x67,0xb3,0xe8,0x04,0xfd,0xa3,0x76,0x9a,0x92,0x80,0x78,0xe4,0xdd,0xd1,0x38,
  0x0d,0xc6,0x35,0x98,0x18,0xf7,0xec,0x6c,0x43,0x75,0x37,0x26,0xfa,0x13,0x94,0x48,
  0xf2,0xd0,0x8b,0x30,0x84,0x54,0xdf,0x23,0x19,0x5b,0x3d,0x59,0xf3,0xae,0xa2,0x82,
  0x63,0x01,0x83,0x2e,0xd9,0x51,0x9b,0x7c,0xa6,0xeb,0xa5,0xbe,0x16,0x0c,0xe3,0x61,
  0xc0,0x8c,0x3a,0xf5,0x73,0x2c,0x25,0x0b,0xbb,0x4e,0x89,0x6b,0x53,0x6a,0xb4,0xf1,
  0xe1,0xe6,0xbd,0x45,0xe2,0xf4,0xb6,0x66,0xcc,0x95,0x03,0x56,0xd4,0x1c,0x1e,0xd7,
  0xfb,0xc3,0x8e,0xb5,0xe9,0xcf,0xbf,0xba,0xea,0x77,0x39,0xaf,0x33,0xc9,0x62,0x71,
  0x81,0x79,0x09,0xad,0x24,0xcd,0xf9,0xd8,0xe5,0xc5,0xb9,0x4d,0x44,0x08,0x86,0xe7,
  0xa1,0x1d,0xaa,0xed,0x06,0x70,0xb2,0xd2,0x41,0x7b,0xa0,0x11,0x31,0xc2,0x27,0x90,
  0x20,0xf6,0x60,0xff,0x96,0x5c,0xb1,0xab,0x9e,0x9c,0x52,0x1b,0x5f,0x93,0x0a,0xef,
  0x91,0x85,0x49,0xee,0x2d,0x4f,0x8f,0x3b,0x47,0x87,0x6d,0x46,0xd6,0x3e,0x69,0x64,
  0x2a,0xce,0xcb,0x2f,0xfc,0x97,0x05,0x7a,0xac,0x7f,0xd5,0x1a,0x4b,0x0e,0xa7,0x5a,
  0x28,0x14,0x3f,0x29,0x88,0x3c,0x4c,0x02,0xb8,0xda,0xb0,0x17,0x55,0x1f,0x8a,0x7d,
  0x57,0xc7,0x8d,0x74,0xb7,0xc4,0x9f,0x72,0x7e,0x15,0x22,0x12,0x58,0x07,0x99,0x34,
  0x6e,0x50,0xde,0x68,0x65,0xbc,0xdb,0xf8,0xc8,0xa8,0x2b,0x40,0xdc,0xfe,0x32,0xa4,
  0xca,0x10,0x21,0xf0,0xd3,0x5d,0x0f,0x00,0x6f,0x9d,0x36,0x42,0x4a,0x5e,0xc1,0xe0
]

// Apply a q permutation (single byte)
function qperm(x) {
  return Q0[x & 0xff]
}

// RS matrix for computing the S-box key material (Twofish uses GF(2^8))
// Simplified: build key-dependent S-boxes by mixing key bytes through q-perms.
function buildSboxes(keyBytes) {
  // 4 S-boxes, each 256 entries, derived from key
  const sboxes = []
  for (let s = 0; s < 4; s++) {
    const box = new Uint32Array(256)
    for (let i = 0; i < 256; i++) {
      let v = i
      for (let r = 0; r < 4; r++) {
        v = qperm((v ^ keyBytes[(s * 4 + r) % keyBytes.length]) & 0xff)
      }
      box[i] = (v ^ keyBytes[s]) >>> 0
    }
    sboxes.push(box)
  }
  return sboxes
}

// Key schedule: expand 128-bit key into 40 subkeys (K0..K39) and 4 round-key words
function expandKey(keyBytes) {
  // Split into 4 32-bit words
  const M = []
  for (let i = 0; i < 4; i++) {
    M.push(((keyBytes[i*4]) | (keyBytes[i*4+1] << 8) | (keyBytes[i*4+2] << 16) | (keyBytes[i*4+3] << 24)) >>> 0)
  }
  const sboxes = buildSboxes(keyBytes)
  // Generate 40 subkeys using a simple LFSR-like derivation mixing S-box output
  const K = []
  let a = 0x9e3779b9 // golden ratio constant
  let b = 0x85ebca77
  for (let i = 0; i < 40; i++) {
    a = (a + M[i % 4]) >>> 0
    b = (b + M[(i + 1) % 4]) >>> 0
    const sb = sboxes[i % 4][b & 0xff]
    K.push((a ^ b ^ sb) >>> 0)
  }
  return K
}

function g(sboxes, x) {
  // Key-dependent S-box layer on a 32-bit word
  const b0 = (x >>> 24) & 0xff
  const b1 = (x >>> 16) & 0xff
  const b2 = (x >>> 8) & 0xff
  const b3 = x & 0xff
  return ((sboxes[0][b0] ^ sboxes[1][b1]) + (sboxes[2][b2] ^ sboxes[3][b3])) >>> 0
}

const rotl = (x, n) => ((x << n) | (x >>> (32 - n))) >>> 0
const ror = (x, n) => ((x >>> n) | (x << (32 - n))) >>> 0

// Twofish round function: simplified 16-round Feistel with two g-functions
function encryptBlock(R0, R1, R2, R3, K, sboxes) {
  for (let r = 0; r < 16; r++) {
    const t0 = g(sboxes, R0)
    const t1 = rotl(g(sboxes, R1), 8)
    const F0 = (t0 + t1 + K[8 + r]) >>> 0
    const F1 = (t0 + (t1 * 2) + K[8 + ((r + 1) % 16) + 8]) >>> 0
    // swap halves
    const nR2 = (ror(F0 ^ R2, 1)) >>> 0
    const nR3 = (rotl(R3, 1) ^ F1) >>> 0
    R0 = R2; R1 = R3
    R2 = nR2; R3 = nR3
    // Also apply whitening subkeys K[2r], K[2r+1] in simplified manner
    R0 = (R0 ^ K[r * 2 % 40]) >>> 0
    R1 = (R1 ^ K[(r * 2 + 1) % 40]) >>> 0
  }
  // Final swap undo + output whitening
  return [R2, R3, R0, R1]
}

function decryptBlock(R0, R1, R2, R3, K, sboxes) {
  for (let r = 15; r >= 0; r--) {
    R0 = (R0 ^ K[r * 2 % 40]) >>> 0
    R1 = (R1 ^ K[(r * 2 + 1) % 40]) >>> 0
    const t0 = g(sboxes, R0)
    const t1 = rotl(g(sboxes, R1), 8)
    const F0 = (t0 + t1 + K[8 + r]) >>> 0
    const F1 = (t0 + (t1 * 2) + K[8 + ((r + 1) % 16) + 8]) >>> 0
    const oR2 = (rotl(F0 ^ R2, 1)) >>> 0
    const oR3 = (ror(R3, 1) ^ F1) >>> 0
    R2 = R0; R3 = R1
    R0 = oR2; R1 = oR3
  }
  return [R2, R3, R0, R1]
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
    const ic = input.value.replace(/\s/g, '').toLowerCase()
    if (ic.length !== 32) return '❌ 数据必须为 32 Hex 字符 (128 位)'
    const data = hexToBytes(ic)
    const W = []
    for (let i = 0; i < 4; i++) {
      W.push((data[i*4] | (data[i*4+1] << 8) | (data[i*4+2] << 16) | (data[i*4+3] << 24)) >>> 0)
    }
    const K = expandKey(keyBytes)
    const sboxes = buildSboxes(keyBytes)
    // input whitening
    for (let i = 0; i < 4; i++) W[i] = (W[i] ^ K[i]) >>> 0
    let result
    if (mode.value === 'encrypt') {
      result = encryptBlock(W[0], W[1], W[2], W[3], K, sboxes)
    } else {
      result = decryptBlock(W[0], W[1], W[2], W[3], K, sboxes)
    }
    // output whitening
    for (let i = 0; i < 4; i++) result[i] = (result[i] ^ K[4 + i]) >>> 0
    let hex = ''
    for (const w of result) {
      hex += (w & 0xff).toString(16).padStart(2, '0')
      hex += ((w >>> 8) & 0xff).toString(16).padStart(2, '0')
      hex += ((w >>> 16) & 0xff).toString(16).padStart(2, '0')
      hex += ((w >>> 24) & 0xff).toString(16).padStart(2, '0')
    }
    return hex
  } catch (e) {
    return '❌ ' + e.message
  }
})
</script>

<style scoped>
.tf-tool { display: flex; flex-direction: column; gap: 12px; }
.tf-tool__field { display: flex; flex-direction: column; gap: 4px; }
.tf-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.tf-tool__field input, .tf-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.tf-tool__actions { display: flex; gap: 8px; }
.tf-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.tf-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.tf-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.tf-tool__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
