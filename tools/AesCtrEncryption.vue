<template>
  <h-single-layout>
    <div class="aesctr">
      <div class="aesctr__field"><label>密钥 (文本，自动通过 PBKDF2 派生 256 位)</label><input v-model="password" type="text" spellcheck="false" /></div>
      <div class="aesctr__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文':'Base64 密文 (含 nonce 前缀)'" spellcheck="false"></textarea></div>
      <div class="aesctr__actions">
        <button :class="{active: mode==='encrypt'}" @click="mode='encrypt'">加密 (AES-256-CTR)</button>
        <button :class="{active: mode==='decrypt'}" @click="mode='decrypt'">解密</button>
      </div>
      <div v-if="loading" class="aesctr__loading">处理中...</div>
      <div v-else class="aesctr__output selectable">{{ output }}</div>
      <div class="aesctr__hint">AES-CTR 流式加密，需双方共享相同 nonce。输出为 Base64(16字节 nonce ‖ 密文)。注意：CTR 模式不提供完整性校验</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, watch } from 'vue'
const password = ref('my-aes-ctr-key')
const input = ref('Hello AES-CTR!')
const mode = ref('encrypt')
const output = ref('')
const loading = ref(false)

// Derive a 256-bit AES key from a password via PBKDF2 (SHA-256).
async function deriveKey(pass) {
  const enc = new TextEncoder()
  const keyMaterial = await crypto.subtle.importKey(
    'raw', enc.encode(pass), 'PBKDF2', false, ['deriveKey']
  )
  const salt = enc.encode('supertools-aes-ctr-salt')
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: 100000, hash: 'SHA-256' },
    keyMaterial,
    { name: 'AES-CTR', length: 256 },
    false,
    ['encrypt', 'decrypt']
  )
}

const buf2b64 = (buf) => btoa(String.fromCharCode(...new Uint8Array(buf)))
const b642buf = (str) => Uint8Array.from(atob(str), c => c.charCodeAt(0))

// Async processing with loading state; re-run on any input change.
async function process() {
  if (!input.value) { output.value = ''; return }
  loading.value = true
  try {
    const key = await deriveKey(password.value)
    const enc = new TextEncoder()
    if (mode.value === 'encrypt') {
      // CTR nonce must be 16 bytes (full AES block size) for Web Crypto.
      const nonce = crypto.getRandomValues(new Uint8Array(16))
      const counter = nonce
      const ciphertext = await crypto.subtle.encrypt(
        { name: 'AES-CTR', counter, length: 64 },
        key,
        enc.encode(input.value)
      )
      const combined = new Uint8Array(nonce.length + ciphertext.byteLength)
      combined.set(nonce)
      combined.set(new Uint8Array(ciphertext), nonce.length)
      output.value = buf2b64(combined.buffer)
    } else {
      const data = b642buf(input.value.trim())
      const nonce = data.slice(0, 16)
      const ciphertext = data.slice(16)
      const plain = await crypto.subtle.decrypt(
        { name: 'AES-CTR', counter: nonce, length: 64 },
        key,
        ciphertext
      )
      output.value = new TextDecoder().decode(plain)
    }
  } catch (e) {
    output.value = '❌ ' + e.message
  } finally {
    loading.value = false
  }
}

watch([input, password, mode], process, { immediate: true })
</script>

<style scoped>
.aesctr { display: flex; flex-direction: column; gap: 12px; }
.aesctr__field { display: flex; flex-direction: column; gap: 4px; }
.aesctr__field label { font-size: 12px; color: var(--text-tertiary); }
.aesctr__field input, .aesctr__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.aesctr__actions { display: flex; gap: 8px; }
.aesctr__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.aesctr__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.aesctr__loading, .aesctr__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.aesctr__loading { color: var(--text-tertiary); }
.aesctr__output { background: var(--bg-base); }
.aesctr__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
