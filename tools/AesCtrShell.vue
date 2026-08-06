<template>
  <h-single-layout>
    <div class="aesctr">
      <div class="aesctr__field"><label>密钥 (密码文本，自动派生 256 位)</label><input v-model="password" type="text" /></div>
      <div class="aesctr__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文':'Base64 密文'" spellcheck="false"></textarea></div>
      <div class="aesctr__actions"><button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">AES-CTR 加密</button><button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button></div>
      <div v-if="loading" class="aesctr__loading">处理中...</div>
      <div v-else class="aesctr__output selectable">{{ output }}</div>
      <div class="aesctr__hint">AES-256-CTR 模式，流式加密，密文长度 = 明文长度（无 padding）</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, watch } from 'vue'
const password = ref('my-ctr-key')
const input = ref('Hello AES-CTR!')
const mode = ref('encrypt')
const output = ref('')
const loading = ref(false)

async function deriveKey(pass) {
  const enc = new TextEncoder()
  const km = await crypto.subtle.importKey('raw', enc.encode(pass), 'PBKDF2', false, ['deriveKey'])
  return crypto.subtle.deriveKey(
    { name:'PBKDF2', salt: enc.encode('supertools-ctr'), iterations:100000, hash:'SHA-256' },
    km, { name:'AES-CTR', length:256 }, false, ['encrypt','decrypt']
  )
}
const b2b64 = (b) => btoa(String.fromCharCode(...new Uint8Array(b)))
const b642b = (s) => Uint8Array.from(atob(s), c=>c.charCodeAt(0))

async function process() {
  if (!input.value) { output.value=''; return }
  loading.value = true
  try {
    const key = await deriveKey(password.value)
    const enc = new TextEncoder()
    if (mode.value === 'encrypt') {
      const counter = crypto.getRandomValues(new Uint8Array(16))
      const ct = await crypto.subtle.encrypt({ name:'AES-CTR', counter }, key, enc.encode(input.value))
      const combined = new Uint8Array(counter.length + ct.byteLength)
      combined.set(counter); combined.set(new Uint8Array(ct), counter.length)
      output.value = b2b64(combined.buffer)
    } else {
      const data = b642b(input.value.trim())
      const counter = data.slice(0, 16)
      const ct = data.slice(16)
      const pt = await crypto.subtle.decrypt({ name:'AES-CTR', counter }, key, ct)
      output.value = new TextDecoder().decode(pt)
    }
  } catch(e) { output.value = '❌ ' + e.message }
  finally { loading.value = false }
}
watch([input, password, mode], process, { immediate: true })
</script>
<style scoped>
.aesctr{display:flex;flex-direction:column;gap:12px}
.aesctr__field{display:flex;flex-direction:column;gap:4px}
.aesctr__field label{font-size:12px;color:var(--text-tertiary)}
.aesctr__field input,.aesctr__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.aesctr__actions{display:flex;gap:8px}
.aesctr__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.aesctr__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.aesctr__loading,.aesctr__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.aesctr__loading{color:var(--text-tertiary)}
.aesctr__output{background:var(--bg-base)}
.aesctr__hint{font-size:12px;color:var(--text-tertiary)}
</style>
