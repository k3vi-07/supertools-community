<template>
  <h-single-layout>
    <div class="ts-gen">
      <div class="ts-gen__input">
        <label>Header (JSON)</label>
        <textarea v-model="headerJson" rows="4" class="ts-gen__ta selectable"></textarea>
      </div>
      <div class="ts-gen__input">
        <label>Payload (JSON)</label>
        <textarea v-model="payloadJson" rows="6" class="ts-gen__ta selectable"></textarea>
      </div>
      <div class="ts-gen__input">
        <label>密钥 (Secret)</label>
        <input v-model="secret" type="password" class="ts-gen__field" />
      </div>
      <div class="ts-gen__output">
        <div class="ts-gen__header"><span>生成的 JWT</span><button v-if="token" @click="copy">复制</button></div>
        <textarea v-if="token" v-model="token" rows="4" class="ts-gen__ta selectable" readonly></textarea>
        <div v-else class="ts-gen__hint">填写 Header 和 Payload 后生成</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, watch } from 'vue'
const headerJson = ref('{\n  "alg": "HS256",\n  "typ": "JWT"\n}')
const payloadJson = ref('{\n  "sub": "1234567890",\n  "name": "SuperTools",\n  "iat": ' + Math.floor(Date.now()/1000) + '\n}')
const secret = ref('your-secret-key')
const token = ref('')

// Inline HMAC-SHA256 implementation (no external dependency)
function strToArrayBuffer(str) {
  const buf = new Uint8Array(str.length)
  for (let i = 0; i < str.length; i++) buf[i] = str.charCodeAt(i) & 0xff
  return buf
}
function arrayBufferToBase64(buf) {
  let binary = ''
  const bytes = new Uint8Array(buf)
  for (let i = 0; i < bytes.byteLength; i++) binary += String.fromCharCode(bytes[i])
  return btoa(binary)
}
async function hmacSha256(key, message) {
  const enc = new TextEncoder()
  const cryptoKey = await crypto.subtle.importKey('raw', enc.encode(key), { name: 'HMAC', hash: 'SHA-256' }, false, ['sign'])
  const sig = await crypto.subtle.sign('HMAC', cryptoKey, enc.encode(message))
  return arrayBufferToBase64(sig)
}
function base64UrlEncode(str) {
  return btoa(str).replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '')
}
function base64UrlFromBase64(b64) {
  return b64.replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '')
}

async function generate() {
  try {
    const header = JSON.parse(headerJson.value)
    const payload = JSON.parse(payloadJson.value)
    const h = base64UrlEncode(JSON.stringify(header))
    const p = base64UrlEncode(JSON.stringify(payload))
    const sigB64 = await hmacSha256(secret.value, h + '.' + p)
    const sig = base64UrlFromBase64(sigB64)
    token.value = h + '.' + p + '.' + sig
  } catch (err) {
    token.value = '错误: ' + err.message
  }
}
function copy() { window.$he3?.copyText(token.value); window.$he3?.message.success('已复制') }
watch([headerJson, payloadJson, secret], generate, { immediate: true })
</script>

<style scoped>
.ts-gen { display: flex; flex-direction: column; gap: 16px; }
.ts-gen__input { display: flex; flex-direction: column; gap: 4px; }
.ts-gen__input label { font-size: 12px; color: var(--text-secondary); }
.ts-gen__ta { padding: 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 12px; outline: none; resize: vertical; }
.ts-gen__field { padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; }
.ts-gen__output { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.ts-gen__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.ts-gen__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.ts-gen__hint { padding: 16px; color: var(--text-tertiary); font-size: 13px; }
</style>
