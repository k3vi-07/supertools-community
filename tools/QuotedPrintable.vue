<template>
  <h-single-layout>
    <div class="amf4">
      <div class="amf4__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入 Quoted-Printable'" spellcheck="false"></textarea>
      </div>
      <div class="amf4__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="amf4__output selectable">{{ output }}</div>
      <p class="amf4__hint">Quoted-Printable 编码用于邮件传输，非 ASCII 字符用 =HH 表示，行尾用 = 软换行。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello 世界')
const mode = ref('encode')

const output = computed(() => {
  if (!input.value) return ''
  if (mode.value === 'encode') {
    const bytes = new TextEncoder().encode(input.value)
    return [...bytes].map(b => {
      if (b === 0x0d || b === 0x0a) return String.fromCharCode(b)
      if (b === 61) return '=3D'
      if (b >= 33 && b <= 126) return String.fromCharCode(b)
      return '=' + b.toString(16).toUpperCase().padStart(2, '0')
    }).join('').replace(/\n/g, '\r\n')
  } else {
    const clean = input.value.replace(/=\r?\n/g, '')
    const str = clean.replace(/=([0-9a-fA-F]{2})/g, (_, h) => String.fromCharCode(parseInt(h, 16)))
    const bytes = [...str].map(c => c.charCodeAt(0))
    return new TextDecoder('utf-8').decode(new Uint8Array(bytes))
  }
})
</script>

<style scoped>
.amf4{display:flex;flex-direction:column;gap:12px}
.amf4__field{display:flex;flex-direction:column;gap:4px}
.amf4__field label{font-size:12px;color:var(--text-tertiary)}
.amf4__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.amf4__actions{display:flex;gap:8px}
.amf4__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.amf4__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.amf4__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.amf4__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
