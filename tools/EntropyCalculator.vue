<template>
  <h-single-layout>
    <div class="entropy">
      <textarea v-model="input" class="entropy__input" rows="5" placeholder="输入文本分析熵值..." spellcheck="false"></textarea>
      <div class="entropy__stats">
        <div class="entropy__stat"><div class="entropy__num">{{ shannon.toFixed(2) }}</div><div class="entropy__label">Shannon 熵 (bits/char)</div></div>
        <div class="entropy__stat"><div class="entropy__num">{{ maxEntropy.toFixed(2) }}</div><div class="entropy__label">理论最大熵</div></div>
        <div class="entropy__stat"><div class="entropy__num">{{ (shannon/maxEntropy*100||0).toFixed(0) }}%</div><div class="entropy__label">随机性</div></div>
        <div class="entropy__stat"><div class="entropy__num">{{ uniqueChars }}</div><div class="entropy__label">独立字符数</div></div>
      </div>
      <div class="entropy__bar"><div class="entropy__fill" :style="{width:(shannon/maxEntropy*100||0)+'%'}"></div></div>
      <div class="entropy__dist">
        <div class="entropy__dist-label">字符分布 TOP 15</div>
        <div class="entropy__dist-bars">
          <div v-for="c in topChars" :key="c.ch" class="entropy__bar-item">
            <div class="entropy__bar-char">{{ c.ch === ' ' ? '␣' : c.ch === '\n' ? '↵' : c.ch }}</div>
            <div class="entropy__bar-wrap"><div class="entropy__bar-fill" :style="{height: c.pct+'%'}"></div></div>
            <div class="entropy__bar-pct">{{ c.count }}</div>
          </div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const input = ref('The quick brown fox jumps over the lazy dog')

const analysis = computed(() => {
  const s = input.value
  if (!s) return { shannon:0, max:0, unique:0, dist:{} }
  const freq = {}
  for (const c of s) freq[c] = (freq[c]||0)+1
  const len = s.length
  let entropy = 0
  for (const c in freq) { const p = freq[c]/len; entropy -= p*Math.log2(p) }
  const unique = Object.keys(freq).length
  const max = Math.log2(unique) || 0
  return { shannon:entropy, max, unique, dist:freq }
})

const shannon = computed(() => analysis.value.shannon)
const maxEntropy = computed(() => analysis.value.max)
const uniqueChars = computed(() => analysis.value.unique)

const topChars = computed(() => {
  const dist = analysis.value.dist
  const entries = Object.entries(dist).map(([ch,count])=>({ch,count}))
  entries.sort((a,b)=>b.count-a.count)
  const max = entries[0]?.count || 1
  return entries.slice(0,15).map(e=>({...e,pct:Math.round(e.count/max*100)}))
})
</script>
<style scoped>
.entropy{display:flex;flex-direction:column;gap:12px}
.entropy__input{width:100%;padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;resize:vertical;outline:none}
.entropy__stats{display:grid;grid-template-columns:repeat(4,1fr);gap:8px}
.entropy__stat{display:flex;flex-direction:column;align-items:center;padding:12px 8px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface)}
.entropy__num{font-size:24px;font-weight:700;color:var(--color-primary)}
.entropy__label{font-size:11px;color:var(--text-tertiary);text-align:center;margin-top:4px}
.entropy__bar{height:8px;border-radius:4px;background:var(--bg-active);overflow:hidden}
.entropy__fill{height:100%;background:var(--color-primary);border-radius:4px;transition:width .3s}
.entropy__dist-label{font-size:12px;color:var(--text-tertiary);margin-bottom:8px}
.entropy__dist-bars{display:flex;align-items:flex-end;gap:4px;height:100px}
.entropy__bar-item{display:flex;flex-direction:column;align-items:center;flex:1;max-width:40px;height:100%}
.entropy__bar-char{font-size:12px;font-family:monospace;color:var(--text-secondary);margin-bottom:2px}
.entropy__bar-wrap{flex:1;width:100%;display:flex;align-items:flex-end;background:var(--bg-active);border-radius:3px}
.entropy__bar-fill{width:100%;background:var(--color-primary);border-radius:3px;min-height:2px;transition:height .3s}
.entropy__bar-pct{font-size:10px;color:var(--text-tertiary);margin-top:2px}
</style>
