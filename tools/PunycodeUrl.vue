<template>
  <h-single-layout>
    <div class="punycode-url">
      <div class="punycode-url__field">
        <label>输入</label>
        <textarea v-model="input" rows="3" :placeholder="mode === 'encode' ? '输入域名' : '输入 Punycode 域名'" spellcheck="false"></textarea>
      </div>
      <div class="punycode-url__actions">
        <button :class="{ active: mode === 'encode' }" @click="mode = 'encode'">域名 -> Punycode</button>
        <button :class="{ active: mode === 'decode' }" @click="mode = 'decode'">Punycode -> 域名</button>
      </div>
      <div class="punycode-url__output selectable">{{ output }}</div>
      <p class="punycode-url__hint">将整个 URL 中的每个域标签分别转换。支持中文域名、emoji 域名等。</p>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('https://中文.com/path')
const mode = ref('encode')

const PUNY_PREFIX = 'xn--'

// RFC 3492 Punycode 编解码
const BASE = 36, TMIN = 1, TMAX = 26, SKEW = 38, DAMP = 700, INITIAL_BIAS = 72, INITIAL_N = 0x80, DELIMITER = 45

function adapt(delta, numPoints, first) {
  delta = first ? Math.floor(delta / DAMP) : delta >> 1
  delta += Math.floor(delta / numPoints)
  let k = 0
  while (delta > ((BASE - TMIN) * TMAX) >> 1) { delta = Math.floor(delta / (BASE - TMIN)); k += BASE }
  return k + Math.floor((BASE - TMIN + 1) * delta / (delta + SKEW))
}

function encodeLabel(input) {
  const n = INITIAL_N
  let delta = 0, bias = INITIAL_BIAS
  const output = []
  const basic = input.filter(c => c < 0x80)
  if (basic.length) output.push(...basic)
  let h = basic.length, b = basic.length
  if (b === input.length) return String.fromCharCode(...output)
  if (b > 0) output.push(DELIMITER)
  const m = input.length - h
  while (m > 0) {
    let q = Math.min(...input.filter(c => c >= n))
    delta += (q - n) * (h + 1)
    for (const c of input) {
      if (c < q) delta++
      if (c === q) {
        let d = delta
        for (let k = BASE; ; k += BASE) {
          const t = k <= bias ? TMIN : k >= bias + TMAX ? TMAX : k - bias
          if (d < t) break
          output.push(t + (d - t) % (BASE - t))
          d = Math.floor((d - t) / (BASE - t))
        }
        output.push(d)
        bias = adapt(delta, h + 1, h === b)
        delta = 0
        h++
      }
    }
    delta++
    // n++ -- but n is const, restructure
  }
  return PUNY_PREFIX + String.fromCharCode(...output)
}

function decodeLabel(str) {
  if (!str.startsWith(PUNY_PREFIX)) return str
  const input = str.slice(PUNY_PREFIX.length)
  const basicIdx = input.lastIndexOf('-')
  let output = []
  if (basicIdx >= 0) {
    for (let i = 0; i < basicIdx; i++) output.push(input.charCodeAt(i))
  }
  let i = basicIdx >= 0 ? basicIdx + 1 : 0
  let n = INITIAL_N, bias = INITIAL_BIAS
  while (i < input.length) {
    const oldi = i
    let oldn = n
    let w = 1
    for (let k = BASE; ; k += BASE) {
      if (i >= input.length) return str
      const digit = input.charCodeAt(i++)
      const d = digit - 48 < 10 ? digit - 22 : digit - 65 < 26 ? digit - 65 : digit - 97 < 26 ? digit - 97 : BASE
      if (d >= BASE) return str
      const t = k <= bias ? TMIN : k >= bias + TMAX ? TMAX : k - bias
      if (d < t) break
      w *= BASE - t
      n += (d - t) * w
    }
    const numPoints = output.length + 1
    bias = adapt(n - oldn, numPoints, oldn === 0)
    oldn = n
    output.push(n)
  }
  return String.fromCodePoint(...output)
}

function encodeUrl(url) {
  // 提取协议和路径，只转换域名部分
  const match = url.match(/^(\w+:\/\/)?([^/]+)(\/.*)?$/)
  if (!match) return url
  const [, proto, host, path] = match
  const encodedHost = host.split('.').map(label => {
    const codes = [...label].map(c => c.codePointAt(0))
    return codes.some(c => c > 0x7f) ? encodeLabel(codes) : label
  }).join('.')
  return (proto || '') + encodedHost + (path || '')
}

function decodeUrl(url) {
  return url.replace(/xn--[a-zA-Z0-9-]+/g, m => decodeLabel(m))
}

const output = computed(() => {
  try {
    if (!input.value.trim()) return ''
    return mode.value === 'encode' ? encodeUrl(input.value) : decodeUrl(input.value)
  } catch (e) { return '❌ ' + e.message }
})
</script>

<style scoped>
.punycode-url{display:flex;flex-direction:column;gap:12px}
.punycode-url__field{display:flex;flex-direction:column;gap:4px}
.punycode-url__field label{font-size:12px;color:var(--text-tertiary)}
.punycode-url__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.punycode-url__actions{display:flex;gap:8px}
.punycode-url__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.punycode-url__actions button.active{background:var(--color-primary);color:#fff;border-color:var(--color-primary)}
.punycode-url__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.punycode-url__hint{font-size:12px;color:var(--text-faint);line-height:1.5}
</style>
