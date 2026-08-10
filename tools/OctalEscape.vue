<template>
  <h-single-layout>
    <div class="amf">
      <div class="amf__field">
        <label>输入</label>
        <textarea v-model="input" rows="5" :placeholder="mode === 'encode' ? '输入文本' : '输入八进制转义（\\xxx 格式）'" spellcheck="false"></textarea>
      </div>
      <div class="amf__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">编码</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解码</button>
      </div>
      <div class="amf__output selectable">{{ output }}</div>
      <p class="amf__hint">将非 ASCII 字符转换为 \xxx 八进制转义序列，常用于 C 字符串和日志处理。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('你好 Hello')
const mode = ref('encode')

const output = computed(() => {
  try {
    if (!input.value) return ''
    if (mode.value === 'encode') {
      const bytes = new TextEncoder().encode(input.value)
      return [...bytes].map(b => b > 127 || b < 32 ? '\\' + b.toString(8).padStart(3, '0') : String.fromCharCode(b)).join('')
    } else {
      const str = input.value.replace(/\\([0-7]{1,3})/g, (_, oct) => String.fromCharCode(parseInt(oct, 8)))
      const bytes = [...str].map(c => c.charCodeAt(0))
      return new TextDecoder().decode(new Uint8Array(bytes))
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.amf{display:flex;flex-direction:column;gap:12px}
.amf__field{display:flex;flex-direction:column;gap:4px}
.amf__field label{font-size:12px;color:var(--text-tertiary)}
.amf__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.amf__actions{display:flex;gap:8px}
.amf__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.amf__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.amf__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all;white-space:pre-wrap}
.amf__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
