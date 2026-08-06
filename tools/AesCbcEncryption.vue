<template>
  <h-single-layout>
    <div class="aescbc">
      <div class="aescbc__field"><label>密钥 (文本，自动通过 PBKDF2 派生 256 位)</label><input v-model="password" type="text" spellcheck="false" /></div>
      <div class="aescbc__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文':'Base64 密文 (含 IV 前缀)'" spellcheck="false"></textarea></div>
      <div class="aescbc__actions">
        <button :class="{active: mode==='encrypt'}" @click="mode='encrypt'">加密 (AES-256-CBC + PKCS7)</button>
        <button :class="{active: mode==='decrypt'}" @click="mode='decrypt'">解密</button>
      </div>
      <div v-if="loading" class="aescbc__loading">处理中...</div>
      <div v-else class="aescbc__output selectable">{{ output }}</div>
      <div class="aescbc__hint">AES-CBC 分组加密，PKCS7 填充。输出为 Base64(16字节 IV ‖ 密文)。CBC 模式不提供完整性校验，需要可靠的 IV 传递</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, watch } from 'vue'
const password = ref('my-aes-cbc-key')
const input = ref('Hello AES-CBC!')
const mode = ref('encrypt')
const output = ref('')
const loading = ref(false)

// Derive a 256-bit AES key from a password via PBKDF2 (SHA-256).
async function deriveKey(pass) {
  const enc = new TextEncoder()
  const keyMaterial = await crypto.subtle.importKey(
    'raw', enc.encode(pass), 'PBKDF2', false, ['deriveKey']
  )
  const salt = enc.encode('supertools-aes-cbc-salt')
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: 100000, hash: 'SHA-256' },
    keyMaterial,
    { name: 'AES-CBC', length: 256 },
    false,
    ['encrypt', 'decrypt']
  )
}

const buf2b64 = (buf) => btoa(String.fromCharCode(...new Uint8Array(buf)))
const b642buf = (str) => Uint8Array.from(atob(str), c => c.charCodeAt(0))

// Async processing with loading state; re-run on any input change.
// Web Crypto handles PKCS7 padding automatically for AES-CBC.
async function process() {
  if (!input.value) { output.value = ''; return }
  loading.value = true
  try {
    const key = await deriveKey(password.value)
    const enc = new TextEncoder()
    if (mode.value === 'encrypt') {
      // CBC IV must be exactly 16 bytes (one AES block).
      const iv = crypto.getRandomValues(new Uint8Array(16))
      const ciphertext = await crypto.subtle.encrypt(
        { name: 'AES-CBC', iv },
        key,
        enc.encode(input.value)
      )
      const combined = new Uint8Array(iv.length + ciphertext.byteLength)
      combined.set(iv)
      combined.set(new Uint8Array(ciphertext), iv.length)
      output.value = buf2b64(combined.buffer)
    } else {
      const data = b642buf(input.value.trim())
      const iv = data.slice(0, 16)
      const ciphertext = data.slice(16)
      const plain = await crypto.subtle.decrypt(
        { name: 'AES-CBC', iv },
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
.aescbc { display: flex; flex-direction: column; gap: 12px; }
.aescbc__field { display: flex; flex-direction: column; gap: 4px; }
.aescbc__field label { font-size: 12px; color: var(--text-tertiary); }
.aescbc__field input, .aescbc__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.aescbc__actions { display: flex; gap: 8px; }
.aescbc__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.aescbc__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.aescbc__loading, .aescbc__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.aescbc__loading { color: var(--text-tertiary); }
.aescbc__output { background: var(--bg-base); }
.aescbc__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
