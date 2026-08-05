<template>
  <h-single-layout>
    <div class="hmac-pro">
      <div class="hmac-pro__field">
        <label>密钥</label>
        <input v-model="key" type="text" spellcheck="false" />
      </div>
      <div class="hmac-pro__field">
        <label>消息</label>
        <textarea v-model="message" rows="3" spellcheck="false"></textarea>
      </div>
      <div class="hmac-pro__field">
        <label>算法</label>
        <select v-model="algo">
          <option value="SHA-1">SHA-1 (HMAC-SHA1)</option>
          <option value="SHA-256">SHA-256 (HMAC-SHA256)</option>
          <option value="SHA-384">SHA-384 (HMAC-SHA384)</option>
          <option value="SHA-512">SHA-512 (HMAC-SHA512)</option>
        </select>
      </div>
      <div v-if="result" class="hmac-pro__results">
        <div class="hmac-pro__result">
          <span>Hex</span>
          <code class="selectable">{{ hexResult }}</code>
          <button @click="copy(hexResult)">复制</button>
        </div>
        <div class="hmac-pro__result">
          <span>Base64</span>
          <code class="selectable">{{ b64Result }}</code>
          <button @click="copy(b64Result)">复制</button>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, watch } from 'vue'

const key = ref('secret-key-2024')
const message = ref('Hello HMAC!')
const algo = ref('SHA-256')
const hexResult = ref('')
const b64Result = ref('')

const result = ref(true)

function bufToHex(buf) {
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('')
}
function bufToB64(buf) {
  let binary = ''
  const bytes = new Uint8Array(buf)
  for (let i = 0; i < bytes.byteLength; i++) binary += String.fromCharCode(bytes[i])
  return btoa(binary)
}

async function compute() {
  try {
    const enc = new TextEncoder()
    const cryptoKey = await crypto.subtle.importKey('raw', enc.encode(key.value), { name: 'HMAC', hash: algo.value }, false, ['sign'])
    const sig = await crypto.subtle.sign('HMAC', cryptoKey, enc.encode(message.value))
    hexResult.value = bufToHex(sig)
    b64Result.value = bufToB64(sig)
  } catch (e) {
    hexResult.value = 'Error: ' + e.message
    b64Result.value = ''
  }
}

watch([key, message, algo], compute, { immediate: true })

function copy(text) {
  navigator.clipboard?.writeText(text)
}
</script>

<style scoped>
.hmac-pro { display: flex; flex-direction: column; gap: 12px; }
.hmac-pro__field { display: flex; flex-direction: column; gap: 4px; }
.hmac-pro__field label { font-size: 12px; color: var(--text-tertiary); }
.hmac-pro__field input, .hmac-pro__field textarea { padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; resize: vertical; }
.hmac-pro__field select { padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; }
.hmac-pro__results { display: flex; flex-direction: column; gap: 8px; }
.hmac-pro__result { display: flex; align-items: center; gap: 8px; padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); }
.hmac-pro__result span { font-size: 12px; color: var(--text-tertiary); min-width: 50px; }
.hmac-pro__result code { flex: 1; font-family: monospace; font-size: 12px; color: var(--color-primary); word-break: break-all; }
.hmac-pro__result button { padding: 4px 10px; border: 1px solid var(--color-primary); border-radius: 4px; background: transparent; color: var(--color-primary); font-size: 11px; cursor: pointer; }
</style>
