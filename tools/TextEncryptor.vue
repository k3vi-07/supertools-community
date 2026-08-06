<template>
  <h-single-layout>
    <div class="text-enc">
      <div class="text-enc__controls">
        <div class="text-enc__field">
          <label>密钥</label>
          <input v-model="password" type="password" placeholder="输入密钥" />
        </div>
        <div class="text-enc__sep"></div>
        <button class="text-enc__btn" :class="{active: mode === 'encrypt'}" @click="mode = 'encrypt'">加密</button>
        <button class="text-enc__btn" :class="{active: mode === 'decrypt'}" @click="mode = 'decrypt'">解密</button>
      </div>
      <div class="text-enc__cols">
        <div class="text-enc__col">
          <label>{{ mode === 'encrypt' ? '明文' : '密文' }}</label>
          <textarea v-model="input" :placeholder="mode === 'encrypt' ? '输入要加密的文本' : '输入要解密的文本'" spellcheck="false"></textarea>
        </div>
        <div class="text-enc__col">
          <label>{{ mode === 'encrypt' ? '密文' : '明文' }}</label>
          <div class="text-enc__output selectable">{{ output }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const password = ref('my-secret-key')
const input = ref('Hello SuperTools')
const mode = ref('encrypt')

// XOR 加密 + Base64 编码（无需外部库）
function xorEncrypt(text, key) {
  const result = []
  for (let i = 0; i < text.length; i++) {
    result.push(String.fromCharCode(text.charCodeAt(i) ^ key.charCodeAt(i % key.length)))
  }
  return result.join('')
}

function toBase64(str) {
  return btoa(unescape(encodeURIComponent(str)))
}

function fromBase64(str) {
  try {
    return decodeURIComponent(escape(atob(str)))
  } catch {
    throw new Error('无效的 Base64 数据')
  }
}

const output = computed(() => {
  if (!input.value) return ''
  if (!password.value) return '⚠️ 请输入密钥'
  try {
    if (mode.value === 'encrypt') {
      return toBase64(xorEncrypt(input.value, password.value))
    } else {
      return xorEncrypt(fromBase64(input.value), password.value)
    }
  } catch (err) {
    return '❌ ' + err.message
  }
})
</script>

<style scoped>
.text-enc { display: flex; flex-direction: column; gap: 12px; }
.text-enc__controls { display: flex; align-items: flex-end; gap: 8px; }
.text-enc__field { display: flex; flex-direction: column; gap: 4px; }
.text-enc__field label { font-size: 12px; color: var(--text-tertiary); }
.text-enc__field input { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); width: 200px; }
.text-enc__sep { flex: 1; }
.text-enc__btn { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.text-enc__btn.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.text-enc__cols { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.text-enc__col { display: flex; flex-direction: column; gap: 4px; }
.text-enc__col label { font-size: 12px; color: var(--text-tertiary); }
.text-enc__col textarea { width: 100%; min-height: 150px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; resize: vertical; outline: none; }
.text-enc__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 150px; white-space: pre-wrap; word-break: break-all; overflow: auto; }
</style>
