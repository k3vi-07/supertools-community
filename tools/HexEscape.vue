<template>
  <h-single-layout>
    <div class="amf2">
      <div class="amf2__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入 \\x 转义序列'" spellcheck="false"></textarea>
      </div>
      <div class="amf2__controls">
        <label><input type="checkbox" v-model="upper"> 大写</label>
        <label><input type="checkbox" v-model="allChars"> 全部字符</label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="amf2__output selectable">{{ output }}</div>
      <p class="amf2__hint">\xHH 十六进制转义，常用于 C/Python/Shell 字符串中表示任意字节。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello 世界')
const mode = ref('encode')
const upper = ref(true)
const allChars = ref(false)

const output = computed(() => {
  if (!input.value) return ''
  if (mode.value === 'encode') {
    const bytes = new TextEncoder().encode(input.value)
    const prefix = upper.value ? '\\X' : '\\x'
    return [...bytes].map(b => {
      if (!allChars.value && b >= 32 && b <= 126 && b !== 0x5c) return String.fromCharCode(b)
      return prefix + b.toString(16).padStart(2, '0').toUpperCase()
    }).join('')
  } else {
    const str = input.value.replace(/\\x([0-9a-fA-F]{2})/gi, (_, h) => String.fromCharCode(parseInt(h, 16)))
    const bytes = [...str].map(c => c.charCodeAt(0))
    return new TextDecoder().decode(new Uint8Array(bytes))
  }
})
</script>

<style scoped>
.amf2{display:flex;flex-direction:column;gap:12px}
.amf2__field{display:flex;flex-direction:column;gap:4px}
.amf2__field label{font-size:12px;color:var(--text-tertiary)}
.amf2__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.amf2__controls{display:flex;align-items:center;gap:16px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.amf2__controls label{display:flex;align-items:center;gap:4px;cursor:pointer}
.amf2__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.amf2__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.amf2__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.amf2__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
