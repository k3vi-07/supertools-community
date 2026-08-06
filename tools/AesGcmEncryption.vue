<template>
  <h-single-layout>
    <div class="aesgcm">
      <div class="aesgcm__field"><label>密钥 (文本，自动派生 256 位)</label><input v-model="password" type="text" /></div>
      <div class="aesgcm__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode === 'encrypt' ? '明文' : 'Base64 密文'" spellcheck="false"></textarea></div>
      <div class="aesgcm__actions">
        <button :class="{active: mode==='encrypt'}" @click="mode='encrypt'">加密 (AES-256-GCM)</button>
        <button :class="{active: mode==='decrypt'}" @click="mode='decrypt'">解密</button>
      </div>
      <div v-if="loading" class="aesgcm__loading">处理中...</div>
      <div v-else class="aesgcm__output selectable">{{ output }}</div>
      <div class="aesgcm__hint">AES-GCM 提供加密 + 完整性验证（认证加密），比 CBC 更安全</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, watch } from 'vue'
const password = ref('my-aes-key')
const input = ref('Hello AES-GCM!')
const mode = ref('encrypt')
const output = ref('')
const loading = ref(false)

async function deriveKey(pass) {
  const enc = new TextEncoder()
  const keyMaterial = await crypto.subtle.importKey('raw', enc.encode(pass), 'PBKDF2', false, ['deriveKey'])
  const salt = enc.encode('supertools-aes-salt')
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: 100000, hash: 'SHA-256' },
    keyMaterial,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt', 'decrypt']
  )
}

const buf2b64 = (buf) => btoa(String.fromCharCode(...new Uint8Array(buf)))
const b642buf = (str) => Uint8Array.from(atob(str), c => c.charCodeAt(0))

async function process() {
  if (!input.value) { output.value = ''; return }
  loading.value = true
  try {
    const key = await deriveKey(password.value)
    const enc = new TextEncoder()
    if (mode.value === 'encrypt') {
      const iv = crypto.getRandomValues(new Uint8Array(12))
      const ciphertext = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, key, enc.encode(input.value))
      const combined = new Uint8Array(iv.length + ciphertext.byteLength)
      combined.set(iv); combined.set(new Uint8Array(ciphertext), iv.length)
      output.value = buf2b64(combined.buffer)
    } else {
      const data = b642buf(input.value.trim())
      const iv = data.slice(0, 12)
      const ciphertext = data.slice(12)
      const plain = await crypto.subtle.decrypt({ name: 'AES-GCM', iv }, key, ciphertext)
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
.aesgcm { display: flex; flex-direction: column; gap: 12px; }
.aesgcm__field { display: flex; flex-direction: column; gap: 4px; }
.aesgcm__field label { font-size: 12px; color: var(--text-tertiary); }
.aesgcm__field input, .aesgcm__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.aesgcm__actions { display: flex; gap: 8px; }
.aesgcm__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.aesgcm__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.aesgcm__loading, .aesgcm__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.aesgcm__loading { color: var(--text-tertiary); }
.aesgcm__output { background: var(--bg-base); }
.aesgcm__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
