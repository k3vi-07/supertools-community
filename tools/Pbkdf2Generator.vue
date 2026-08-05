<template>
  <h-single-layout>
    <div class="pbkdf2">
      <div class="pbkdf2__field">
        <label>密码</label>
        <input v-model="password" type="text" spellcheck="false" />
      </div>
      <div class="pbkdf2__field">
        <label>盐 (Salt)</label>
        <input v-model="salt" type="text" spellcheck="false" />
        <button class="pbkdf2__rand" @click="randomSalt">随机</button>
      </div>
      <div class="pbkdf2__params">
        <div class="pbkdf2__param">
          <label>哈希算法</label>
          <select v-model="hashAlgo">
            <option value="SHA-1">SHA-1</option>
            <option value="SHA-256">SHA-256</option>
            <option value="SHA-384">SHA-384</option>
            <option value="SHA-512">SHA-512</option>
          </select>
        </div>
        <div class="pbkdf2__param">
          <label>迭代次数</label>
          <input type="number" v-model.number="iterations" min="1" step="1000" />
        </div>
        <div class="pbkdf2__param">
          <label>输出长度 (字节)</label>
          <input type="number" v-model.number="keyLength" min="8" max="512" />
        </div>
      </div>
      <button class="pbkdf2__btn" @click="derive">生成密钥</button>
      <div v-if="result" class="pbkdf2__result">
        <label>派生密钥 (Hex)</label>
        <div class="pbkdf2__output selectable">{{ result }}</div>
        <div class="pbkdf2__result-actions">
          <button @click="copy(result)">复制 Hex</button>
          <button @click="copy(b64Result)">复制 Base64</button>
        </div>
      </div>
      <div v-if="error" class="pbkdf2__error">{{ error }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const password = ref('mypassword123')
const salt = ref('s4lty_salt_2024')
const hashAlgo = ref('SHA-256')
const iterations = ref(100000)
const keyLength = ref(32)
const result = ref('')
const b64Result = ref('')
const error = ref('')

function randomSalt() {
  const arr = new Uint8Array(16)
  crypto.getRandomValues(arr)
  salt.value = Array.from(arr).map(b => b.toString(16).padStart(2, '0')).join('')
}

function bufToHex(buf) {
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('')
}
function bufToB64(buf) {
  let binary = ''
  const bytes = new Uint8Array(buf)
  for (let i = 0; i < bytes.byteLength; i++) binary += String.fromCharCode(bytes[i])
  return btoa(binary)
}

async function derive() {
  error.value = ''
  try {
    const enc = new TextEncoder()
    const keyMaterial = await crypto.subtle.importKey('raw', enc.encode(password.value), 'PBKDF2', false, ['deriveBits'])
    const bits = await crypto.subtle.deriveBits(
      { name: 'PBKDF2', salt: enc.encode(salt.value), iterations: iterations.value, hash: hashAlgo.value },
      keyMaterial,
      keyLength.value * 8
    )
    result.value = bufToHex(bits)
    b64Result.value = bufToB64(bits)
  } catch (e) {
    error.value = e.message
  }
}

function copy(text) {
  navigator.clipboard?.writeText(text)
}
</script>

<style scoped>
.pbkdf2 { display: flex; flex-direction: column; gap: 12px; }
.pbkdf2__field { display: flex; align-items: center; gap: 8px; }
.pbkdf2__field label { font-size: 13px; color: var(--text-secondary); min-width: 70px; }
.pbkdf2__field input { flex: 1; padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; }
.pbkdf2__rand { padding: 6px 12px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.pbkdf2__params { display: flex; gap: 12px; }
.pbkdf2__param { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.pbkdf2__param label { font-size: 12px; color: var(--text-tertiary); }
.pbkdf2__param select, .pbkdf2__param input { padding: 6px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; outline: none; }
.pbkdf2__btn { padding: 10px 24px; border: none; border-radius: 8px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.pbkdf2__result { display: flex; flex-direction: column; gap: 6px; }
.pbkdf2__result label { font-size: 12px; color: var(--text-tertiary); }
.pbkdf2__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.pbkdf2__result-actions { display: flex; gap: 8px; }
.pbkdf2__result-actions button { padding: 4px 12px; border: 1px solid var(--color-primary); border-radius: 4px; background: transparent; color: var(--color-primary); font-size: 12px; cursor: pointer; }
.pbkdf2__error { padding: 8px 12px; border-radius: 6px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
</style>
