<template>
  <h-single-layout>
    <div class="esc-pro">
      <div class="esc-pro__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" placeholder="输入文本..." spellcheck="false"></textarea>
      </div>
      <div class="esc-pro__modes">
        <button v-for="m in modes" :key="m.id" :class="{active: mode === m.id}" @click="mode = m.id">{{ m.name }}</button>
      </div>
      <div class="esc-pro__dir">
        <button :class="{active: dir === 'encode'}" @click="dir='encode'">编码</button>
        <button :class="{active: dir === 'decode'}" @click="dir='decode'">解码</button>
      </div>
      <div class="esc-pro__output-section">
        <div class="esc-pro__output selectable">{{ output }}</div>
        <button class="esc-pro__copy" @click="copy">复制</button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('Hello\nWorld\t测试')
const mode = ref('c')
const dir = ref('encode')

const modes = [
  { id: 'c', name: 'C/Java' },
  { id: 'python', name: 'Python' },
  { id: 'json', name: 'JSON' },
  { id: 'sql', name: 'SQL' },
  { id: 'xml', name: 'XML' },
  { id: 'csv', name: 'CSV' },
]

const ESCAPES = { '\n': '\\n', '\r': '\\r', '\t': '\\t', '\\': '\\\\', '\0': '\\0', '\b': '\\b', '\f': '\\f', '\v': '\\v' }
const UNESCAPES = { '\\n': '\n', '\\r': '\r', '\\t': '\t', '\\\\': '\\', '\\0': '\0', '\\b': '\b', '\\f': '\f', '\\v': '\v', "\\'": "'", '\\"': '"' }

function encode(str, m) {
  let s = str
  if (m === 'json') return JSON.stringify(str)
  if (m === 'sql') s = s.replace(/'/g, "''")
  if (m === 'xml') return s.replace(/[<>&"']/g, c => ({'<':'&lt;','>':'&gt;','&':'&amp;','"':'&quot;',"'":'&apos;'}[c]))
  if (m === 'csv') return s.split('\n').map(line => /[",\n]/.test(line) ? '"' + line.replace(/"/g, '""') + '"' : line).join('\n')
  // C/Java/Python
  s = s.replace(/[\n\r\t\\\0\b\f\v]/g, c => ESCAPES[c])
  if (m === 'c') s = s.replace(/"/g, '\\"')
  if (m === 'python') s = s.replace(/'/g, "\\'").replace(/"/g, '\\"')
  return s
}

function decode(str, m) {
  if (m === 'json') { try { return JSON.parse(str) } catch { return '❌ 无效 JSON' } }
  if (m === 'sql') return str.replace(/''/g, "'")
  if (m === 'xml') return str.replace(/&lt;/g,'<').replace(/&gt;/g,'>').replace(/&amp;/g,'&').replace(/&quot;/g,'"').replace(/&apos;/g,"'")
  if (m === 'csv') {
    return str.split('\n').map(line => {
      if (line.startsWith('"') && line.endsWith('"')) return line.slice(1,-1).replace(/""/g, '"')
      return line
    }).join('\n')
  }
  return str.replace(/\\[nrt\\0bfv'"]/g, m => UNESCAPES[m] || m)
}

const output = computed(() => {
  try {
    if (!input.value === undefined || input.value === '') return ''
    return dir.value === 'encode' ? encode(input.value, mode.value) : decode(input.value, mode.value)
  } catch (e) { return '❌ ' + e.message }
})

function copy() { window.$he3?.copyText(output.value); window.$he3?.message.success('已复制') }
</script>

<style scoped>
.esc-pro{display:flex;flex-direction:column;gap:12px}
.esc-pro__field{display:flex;flex-direction:column;gap:4px}
.esc-pro__field label{font-size:12px;color:var(--text-tertiary)}
.esc-pro__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.esc-pro__modes{display:flex;gap:6px;flex-wrap:wrap}
.esc-pro__modes button{padding:6px 12px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer;font-size:12px}
.esc-pro__modes button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.esc-pro__dir{display:flex;gap:6px}
.esc-pro__dir button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.esc-pro__dir button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.esc-pro__output-section{position:relative}
.esc-pro__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:80px;white-space:pre-wrap;word-break:break-all}
.esc-pro__copy{position:absolute;top:8px;right:8px;padding:4px 10px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer;font-size:12px}
.esc-pro__copy:hover{border-color:var(--color-primary);color:var(--color-primary)}
</style>
