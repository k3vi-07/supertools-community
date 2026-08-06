<template>
  <h-single-layout>
    <div class="rc4-tool">
      <div class="rc4-tool__field"><label>密钥</label><input v-model="key" placeholder="输入密钥" /></div>
      <div class="rc4-tool__field"><label>输入</label><textarea v-model="input" rows="4" placeholder="明文或密文(Hex)..." spellcheck="false"></textarea></div>
      <div class="rc4-tool__actions">
        <button @click="mode = 'encrypt'" :class="{active: mode === 'encrypt'}">加密 → Hex</button>
        <button @click="mode = 'decrypt'" :class="{active: mode === 'decrypt'}">Hex → 解密</button>
      </div>
      <div class="rc4-tool__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const key = ref('secret')
const input = ref('Hello RC4')
const mode = ref('encrypt')

function rc4(key, data) {
  const s = Array.from({ length: 256 }, (_, i) => i)
  let j = 0
  for (let i = 0; i < 256; i++) {
    j = (j + s[i] + key.charCodeAt(i % key.length)) & 0xFF
    ;[s[i], s[j]] = [s[j], s[i]]
  }
  const result = []
  let i2 = 0, j2 = 0
  for (let k = 0; k < data.length; k++) {
    i2 = (i2 + 1) & 0xFF
    j2 = (j2 + s[i2]) & 0xFF
    ;[s[i2], s[j2]] = [s[j2], s[i2]]
    result.push(data.charCodeAt(k) ^ s[(s[i2] + s[j2]) & 0xFF])
  }
  return result
}

const output = computed(() => {
  if (!input.value) return ''
  try {
    if (mode.value === 'encrypt') {
      const bytes = rc4(key.value, input.value)
      return bytes.map(b => b.toString(16).padStart(2, '0')).join('')
    } else {
      const hex = input.value.replace(/\s/g, '')
      const bytes = []
      for (let i = 0; i < hex.length; i += 2) bytes.push(parseInt(hex.substr(i, 2), 16))
      const decrypted = bytes.map((b, idx) => String.fromCharCode(b)).join('')
      const result = rc4(key.value, decrypted)
      return result.map(b => String.fromCharCode(b)).join('')
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.rc4-tool { display: flex; flex-direction: column; gap: 12px; }
.rc4-tool__field { display: flex; flex-direction: column; gap: 4px; }
.rc4-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.rc4-tool__field input, .rc4-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.rc4-tool__actions { display: flex; gap: 8px; }
.rc4-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.rc4-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.rc4-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
</style>
