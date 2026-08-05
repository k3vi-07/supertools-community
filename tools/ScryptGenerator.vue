<template>
  <h-single-layout>
    <div class="scrypt-gen">
      <div class="scrypt-gen__field">
        <label>密码</label>
        <input v-model="password" type="text" spellcheck="false" />
      </div>
      <div class="scrypt-gen__field">
        <label>盐 (Salt)</label>
        <input v-model="salt" type="text" spellcheck="false" />
        <button class="scrypt-gen__rand" @click="randomSalt">随机</button>
      </div>
      <div class="scrypt-gen__params">
        <div class="scrypt-gen__param">
          <label>N (CPU/内存成本)</label>
          <select v-model.number="N">
            <option :value="16">16 (2^4)</option>
            <option :value="128">128 (2^7)</option>
            <option :value="1024">1024 (2^10)</option>
            <option :value="4096">4096 (2^12)</option>
            <option :value="16384">16384 (2^14)</option>
          </select>
        </div>
        <div class="scrypt-gen__param">
          <label>r (块大小)</label>
          <input type="number" v-model.number="r" min="1" max="32" />
        </div>
        <div class="scrypt-gen__param">
          <label>p (并行度)</label>
          <input type="number" v-model.number="p" min="1" max="8" />
        </div>
        <div class="scrypt-gen__param">
          <label>输出长度 (字节)</label>
          <input type="number" v-model.number="dkLen" min="16" max="128" />
        </div>
      </div>
      <div class="scrypt-gen__info-box">
        内存消耗: <strong>{{ formatBytes(N * r * 128) }}</strong> ·
        复杂度: <strong>{{ N }}/{{ r }}/{{ p }}</strong>
      </div>
      <button class="scrypt-gen__btn" @click="derive" :disabled="computing">{{ computing ? '计算中...' : '生成密钥' }}</button>
      <div v-if="result" class="scrypt-gen__result">
        <label>派生密钥 (Hex)</label>
        <div class="scrypt-gen__output selectable">{{ result }}</div>
        <button @click="copyResult">复制</button>
      </div>
      <div v-if="error" class="scrypt-gen__error">{{ error }}</div>
      <div class="scrypt-gen__info">
        scrypt 由 Colin Percival 设计，专为抵抗硬件加速攻击（GPU/ASIC），被 Litecoin 和 Tarsnap 使用
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const password = ref('mypassword')
const salt = ref('NaCl_salt_2024')
const N = ref(1024)
const r = ref(8)
const p = ref(1)
const dkLen = ref(32)
const result = ref('')
const computing = ref(false)
const error = ref('')

function randomSalt() {
  const arr = new Uint8Array(16)
  crypto.getRandomValues(arr)
  salt.value = Array.from(arr).map(b => b.toString(16).padStart(2, '0')).join('')
}

function formatBytes(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / 1048576).toFixed(2) + ' MB'
}

function bufToHex(buf) {
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('')
}

// ===== PBKDF2 as building block for scrypt =====
async function pbkdf2(passwordBytes, saltBytes, iterations, keyLen) {
  const keyMaterial = await crypto.subtle.importKey('raw', passwordBytes, 'PBKDF2', false, ['deriveBits'])
  const bits = await crypto.subtle.deriveBits(
    { name: 'PBKDF2', salt: saltBytes, iterations, hash: 'SHA-256' },
    keyMaterial,
    keyLen * 8
  )
  return new Uint8Array(bits)
}

// ===== salsa20/8 core =====
function rotl(a, b) { return ((a << b) | (a >>> (32 - b))) >>> 0 }

function salsa20_8(B) {
  const x = new Array(16)
  for (let i = 0; i < 16; i++) x[i] = (B[i*4] | (B[i*4+1]<<8) | (B[i*4+2]<<16) | (B[i*4+3]<<24)) >>> 0
  const orig = [...x]
  for (let i = 0; i < 4; i++) {
    x[4] ^= rotl((x[0]+x[12])>>>0, 7); x[8] ^= rotl((x[4]+x[0])>>>0, 9)
    x[12] ^= rotl((x[8]+x[4])>>>0, 13); x[0] ^= rotl((x[12]+x[8])>>>0, 18)
    x[9] ^= rotl((x[5]+x[1])>>>0, 7); x[13] ^= rotl((x[9]+x[5])>>>0, 9)
    x[1] ^= rotl((x[13]+x[9])>>>0, 13); x[5] ^= rotl((x[1]+x[13])>>>0, 18)
    x[14] ^= rotl((x[10]+x[6])>>>0, 7); x[2] ^= rotl((x[14]+x[10])>>>0, 9)
    x[6] ^= rotl((x[2]+x[14])>>>0, 13); x[10] ^= rotl((x[6]+x[2])>>>0, 18)
    x[3] ^= rotl((x[15]+x[11])>>>0, 7); x[7] ^= rotl((x[3]+x[15])>>>0, 9)
    x[11] ^= rotl((x[7]+x[3])>>>0, 13); x[15] ^= rotl((x[11]+x[7])>>>0, 18)
    x[1] ^= rotl((x[0]+x[3])>>>0, 7); x[2] ^= rotl((x[1]+x[0])>>>0, 9)
    x[3] ^= rotl((x[2]+x[1])>>>0, 13); x[0] ^= rotl((x[3]+x[2])>>>0, 18)
    x[6] ^= rotl((x[5]+x[4])>>>0, 7); x[7] ^= rotl((x[6]+x[5])>>>0, 9)
    x[4] ^= rotl((x[7]+x[6])>>>0, 13); x[5] ^= rotl((x[4]+x[7])>>>0, 18)
    x[11] ^= rotl((x[10]+x[9])>>>0, 7); x[8] ^= rotl((x[11]+x[10])>>>0, 9)
    x[9] ^= rotl((x[8]+x[11])>>>0, 13); x[10] ^= rotl((x[9]+x[8])>>>0, 18)
    x[12] ^= rotl((x[15]+x[14])>>>0, 7); x[13] ^= rotl((x[12]+x[15])>>>0, 9)
    x[14] ^= rotl((x[13]+x[12])>>>0, 13); x[15] ^= rotl((x[14]+x[13])>>>0, 18)
  }
  const out = new Uint8Array(64)
  for (let i = 0; i < 16; i++) {
    const v = (x[i] + orig[i]) >>> 0
    out[i*4] = v & 0xff; out[i*4+1] = (v>>8)&0xff; out[i*4+2] = (v>>16)&0xff; out[i*4+3] = (v>>24)&0xff
  }
  return out
}

function blockMix(B, r) {
  const X = B.slice((2*r - 1) * 64, (2*r) * 64)
  const Y = new Uint8Array(128 * r)
  for (let i = 0; i < 2 * r; i++) {
    const block = B.slice(i * 64, (i + 1) * 64)
    for (let j = 0; j < 64; j++) X[j] ^= block[j]
    const new_x = salsa20_8(X)
    X.set(new_x)
    if (i % 2 === 0) {
      Y.set(new_x, (i / 2) * 64)
    } else {
      Y.set(new_x, (r + (i - 1) / 2) * 64)
    }
  }
  return Y
}

function integerify(B, r) {
  const offset = (2 * r - 1) * 64
  return (B[offset] | (B[offset+1]<<8) | (B[offset+2]<<16) | (B[offset+3]<<24)) >>> 0
}

function smix(B, r, N, V, X) {
  for (let i = 0; i < N; i++) {
    V.set(X, i * 128 * r)
    const mixed = blockMix(X, r)
    X.set(mixed)
  }
  for (let i = 0; i < N; i++) {
    const j = integerify(X, r) % N
    for (let k = 0; k < 128 * r; k++) X[k] ^= V[j * 128 * r + k]
    const mixed = blockMix(X, r)
    X.set(mixed)
  }
  return X
}

async function scrypt(passwordBytes, saltBytes, N, r, p, dkLen) {
  const MFLen = 128 * r
  const BLen = MFLen * p
  const B = await pbkdf2(passwordBytes, saltBytes, 1, BLen)

  const V = new Uint8Array(N * MFLen)

  for (let i = 0; i < p; i++) {
    const Xi = B.slice(i * MFLen, (i + 1) * MFLen)
    const X = new Uint8Array(Xi)
    smix(X, r, N, V, X)
    B.set(X, i * MFLen)
  }

  return pbkdf2(passwordBytes, B, 1, dkLen)
}

async function derive() {
  error.value = ''
  computing.value = true
  try {
    const enc = new TextEncoder()
    const result_bytes = await scrypt(enc.encode(password.value), enc.encode(salt.value), N.value, r.value, p.value, dkLen.value)
    result.value = bufToHex(result_bytes)
  } catch (e) {
    error.value = e.message
  } finally {
    computing.value = false
  }
}

function copyResult() {
  navigator.clipboard?.writeText(result.value)
}
</script>

<style scoped>
.scrypt-gen { display: flex; flex-direction: column; gap: 12px; }
.scrypt-gen__field { display: flex; align-items: center; gap: 8px; }
.scrypt-gen__field label { font-size: 13px; color: var(--text-secondary); min-width: 70px; }
.scrypt-gen__field input { flex: 1; padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; }
.scrypt-gen__rand { padding: 6px 12px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.scrypt-gen__params { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.scrypt-gen__param { display: flex; flex-direction: column; gap: 4px; }
.scrypt-gen__param label { font-size: 11px; color: var(--text-tertiary); }
.scrypt-gen__param select, .scrypt-gen__param input { padding: 6px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; outline: none; }
.scrypt-gen__info-box { padding: 8px 12px; border-radius: 6px; background: var(--bg-base); font-size: 12px; color: var(--text-secondary); }
.scrypt-gen__info-box strong { color: var(--color-primary); }
.scrypt-gen__btn { padding: 10px 24px; border: none; border-radius: 8px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.scrypt-gen__btn:disabled { opacity: 0.5; cursor: wait; }
.scrypt-gen__result { display: flex; flex-direction: column; gap: 4px; }
.scrypt-gen__result label { font-size: 12px; color: var(--text-tertiary); }
.scrypt-gen__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.scrypt-gen__result button { align-self: flex-start; padding: 4px 12px; border: none; border-radius: 4px; background: var(--color-primary); color: white; font-size: 12px; cursor: pointer; }
.scrypt-gen__error { padding: 8px 12px; border-radius: 6px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
.scrypt-gen__info { padding: 8px 12px; border-radius: 6px; background: var(--bg-base); font-size: 12px; color: var(--text-tertiary); }
</style>
