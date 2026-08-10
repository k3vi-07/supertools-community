<template>
  <h-single-layout>
    <div class="amf3">
      <div class="amf3__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入 &#xxx; 格式'" spellcheck="false"></textarea>
      </div>
      <div class="amf3__controls">
        <label><input type="radio" v-model="format" value="dec" :disabled="mode==='decode'"> 十进制</label>
        <label><input type="radio" v-model="format" value="hex" :disabled="mode==='decode'"> 十六进制</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="amf3__output selectable">{{ output }}</div>
      <p class="amf3__hint">XML 数字字符引用，将任意字符表示为 &#DD; 或 &#xHH; 格式。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello 世界')
const mode = ref('encode')
const format = ref('dec')

const output = computed(() => {
  if (!input.value) return ''
  if (mode.value === 'encode') {
    return [...input.value].map(ch => {
      const code = ch.codePointAt(0)
      return format.value === 'dec' ? `&#${code};` : `&#x${code.toString(16).toUpperCase()};`
    }).join('')
  } else {
    return input.value
      .replace(/&#x([0-9a-fA-F]+);/g, (_, h) => String.fromCodePoint(parseInt(h, 16)))
      .replace(/&#(\d+);/g, (_, n) => String.fromCodePoint(Number(n)))
  }
})
</script>

<style scoped>
.amf3{display:flex;flex-direction:column;gap:12px}
.amf3__field{display:flex;flex-direction:column;gap:4px}
.amf3__field label{font-size:12px;color:var(--text-tertiary)}
.amf3__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.amf3__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.amf3__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.amf3__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.amf3__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.amf3__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.amf3__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
