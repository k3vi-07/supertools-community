<template>
  <h-single-layout>
    <div class="html-entity">
      <div class="html-entity__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入文本' : '输入 HTML 实体'" spellcheck="false"></textarea>
      </div>
      <div class="html-entity__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="html-entity__modes">
        <label><input type="radio" v-model="format" value="named" :disabled="mode==='decode'"> 命名实体</label>
        <label><input type="radio" v-model="format" value="decimal" :disabled="mode==='decode'"> 十进制</label>
        <label><input type="radio" v-model="format" value="hex" :disabled="mode==='decode'"> 十六进制</label>
      </div>
      <div class="html-entity__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello <World> & "Friends" © 2026')
const mode = ref('encode')
const format = ref('named')

const NAMED = { '<': '&lt;', '>': '&gt;', '&': '&amp;', '"': '&quot;', "'": '&apos;', '©': '&copy;', '®': '&reg;', '™': '&trade;', '€': '&euro;', '£': '&pound;', '¥': '&yen;', '¢': '&cent;', '§': '&sect;', '¶': '&para;', '°': '&deg;', '±': '&plusmn;', '×': '&times;', '÷': '&divide;', '«': '&laquo;', '»': '&raquo;', '•': '&bull;', '…': '&hellip;', '—': '&mdash;', '–': '&ndash;', ''': '&lsquo;', ''': '&rsquo;', '"': '&ldquo;', '"': '&rdquo;', ' ': '&nbsp;', '¡': '&iexcl;', '¿': '&iquest;' }
const REVERSE_NAMED = {}
for (const [k, v] of Object.entries(NAMED)) REVERSE_NAMED[v] = k

function encode(str) {
  if (format.value === 'named') {
    return str.replace(/[<>&"'©®™€£¥¢§¶°±×÷«»•…—–''"" ¡¿]/g, c => NAMED[c] || c)
  }
  return str.split('').map(c => {
    const code = c.charCodeAt(0)
    if (code > 127 || /[<>&"']/.test(c)) {
      return format.value === 'decimal' ? `&#${code};` : `&#x${code.toString(16)};`
    }
    return c
  }).join('')
}

function decode(str) {
  return str
    .replace(/&[a-zA-Z]+;/g, m => REVERSE_NAMED[m] || m)
    .replace(/&#(\d+);/g, (_, n) => String.fromCharCode(Number(n)))
    .replace(/&#x([0-9a-fA-F]+);/g, (_, n) => String.fromCharCode(parseInt(n, 16)))
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encode(input.value) : decode(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.html-entity{display:flex;flex-direction:column;gap:12px}
.html-entity__field{display:flex;flex-direction:column;gap:4px}
.html-entity__field label{font-size:12px;color:var(--text-tertiary)}
.html-entity__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.html-entity__actions{display:flex;gap:8px}
.html-entity__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.html-entity__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.html-entity__modes{display:flex;gap:16px;font-size:13px;color:var(--text-secondary)}
.html-entity__modes label{display:flex;align-items:center;gap:4px;cursor:pointer}
.html-entity__modes input{cursor:pointer}
.html-entity__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
</style>
