<template>
  <h-single-layout>
    <div class="base64url">
      <div class="base64url__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 Base64URL 字符串'" spellcheck="false"></textarea>
      </div>
      <div class="base64url__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="base64url__output selectable">{{ output }}</div>
      <p class="base64url__hint">Base64URL 是 URL 安全的 Base64 变体：+ 替换为 -，/ 替换为 _，去掉 = 填充。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello World')
const mode = ref('encode')

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (mode.value === 'encode') {
      return btoa(unescape(encodeURIComponent(input.value)))
        .replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '')
    } else {
      let str = input.value.trim().replace(/-/g, '+').replace(/_/g, '/')
      while (str.length % 4) str += '='
      return decodeURIComponent(escape(atob(str)))
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.base64url{display:flex;flex-direction:column;gap:12px}
.base64url__field{display:flex;flex-direction:column;gap:4px}
.base64url__field label{font-size:12px;color:var(--text-tertiary)}
.base64url__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.base64url__actions{display:flex;gap:8px}
.base64url__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.base64url__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.base64url__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.base64url__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
