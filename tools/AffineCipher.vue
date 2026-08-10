<template>
  <h-single-layout>
    <div class="affine">
      <div class="affine__field">
        <label>输入</label>
        <textarea v-model="input" rows="4" :placeholder="mode === 'encode' ? '输入明文' : '输入密文'" spellcheck="false"></textarea>
      </div>
      <div class="affine__controls">
        <label>a: <input type="number" v-model.number="a" min="1" /></label>
        <label>b: <input type="number" v-model.number="b" min="0" /></label>
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">加密</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">解密</button>
      </div>
      <div class="affine__output selectable">{{ output }}</div>
      <p class="affine__hint">仿射密码: E(x) = (ax + b) mod 26，a 必须与 26 互质。a=1 时退化为凯撒密码。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('AFFINE')
const mode = ref('encode')
const a = ref(5)
const b = ref(8)

function gcd(x, y) { return y === 0 ? x : gcd(y, x % y) }
function modInverse(num, mod) {
  for (let i = 1; i < mod; i++) if ((num * i) % mod === 1) return i
  return -1
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    if (gcd(a.value, 26) !== 1) return '❌ a 必须与 26 互质'
    const clean = input.value.toUpperCase().replace(/[^A-Z]/g, '')
    if (mode.value === 'encode') {
      return clean.split('').map(c => {
        const x = c.charCodeAt(0) - 65
        return String.fromCharCode(((a.value * x + b.value) % 26 + 26) % 26 + 65)
      }).join('')
    } else {
      const inv = modInverse(((a.value % 26) + 26) % 26, 26)
      return clean.split('').map(c => {
        const x = c.charCodeAt(0) - 65
        return String.fromCharCode((inv * ((x - b.value) % 26 + 26)) % 26 + 65)
      }).join('')
    }
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.affine{display:flex;flex-direction:column;gap:12px}
.affine__field{display:flex;flex-direction:column;gap:4px}
.affine__field label{font-size:12px;color:var(--text-tertiary)}
.affine__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.affine__controls{display:flex;align-items:center;gap:12px;font-size:13px;color:var(--text-secondary);flex-wrap:wrap}
.affine__controls input{width:60px;padding:4px 8px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);color:var(--text-primary)}
.affine__controls button{padding:6px 14px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.affine__controls button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.affine__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.affine__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
