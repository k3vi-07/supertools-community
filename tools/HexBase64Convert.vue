<template>
  <h-single-layout>
    <div class="hexb64">
      <div class="hexb64__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encode'?'Hex 字符串':'Base64 字符串'" spellcheck="false"></textarea></div>
      <div class="hexb64__actions"><button :class="{active:mode==='encode'}" @click="mode='encode'">Hex → Base64</button><button :class="{active:mode==='decode'}" @click="mode='decode'">Base64 → Hex</button></div>
      <div class="hexb64__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const input = ref('48656c6c6f')
const mode = ref('encode')
const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (mode.value === 'encode') {
      const hex = input.value.replace(/\s/g,'')
      const bytes = new Uint8Array(hex.length/2)
      for (let i=0;i<bytes.length;i++) bytes[i]=parseInt(hex.substr(i*2,2),16)
      return btoa(String.fromCharCode(...bytes))
    } else {
      const bytes = Uint8Array.from(atob(input.value.trim()), c=>c.charCodeAt(0))
      return [...bytes].map(b=>b.toString(16).padStart(2,'0')).join('')
    }
  } catch(e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.hexb64{display:flex;flex-direction:column;gap:12px}
.hexb64__field{display:flex;flex-direction:column;gap:4px}
.hexb64__field label{font-size:12px;color:var(--text-tertiary)}
.hexb64__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.hexb64__actions{display:flex;gap:8px}
.hexb64__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.hexb64__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.hexb64__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
