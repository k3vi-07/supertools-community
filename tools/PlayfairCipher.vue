<template>
  <h-single-layout>
    <div class="pf">
      <div class="pf__field"><label>密钥（字母，忽略 J）</label><input v-model="key" placeholder="PLAYFAIR EXAMPLE" /></div>
      <div class="pf__field"><label>输入</label><textarea v-model="input" rows="3" spellcheck="false"></textarea></div>
      <div class="pf__actions"><button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">加密</button><button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button></div>
      <div class="pf__matrix">
        <span v-for="(c,i) in matrixFlat" :key="i" class="pf__cell">{{ c }}</span>
      </div>
      <div class="pf__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const key = ref('PLAYFAIR EXAMPLE')
const input = ref('HIDE THE GOLD')
const mode = ref('encrypt')

function buildMatrix(k) {
  const seen = new Set()
  const matrix = []
  const clean = (k.toUpperCase().replace(/[^A-Z]/g,'').replace(/J/g,'I')+'ABCDEFGHIKLMNOPQRSTUVWXYZ')
  for (const c of clean) { if(!seen.has(c)){seen.add(c);matrix.push(c)} }
  return matrix // 25 chars, 5x5
}
function findPos(matrix, ch) { const idx = matrix.indexOf(ch); return [Math.floor(idx/5), idx%5] }

const matrixFlat = computed(() => buildMatrix(key.value))

const output = computed(() => {
  const m = buildMatrix(key.value)
  const clean = input.value.toUpperCase().replace(/[^A-Z]/g,'').replace(/J/g,'I')
  // 分对，相同字母间插 X
  const pairs = []
  for (let i=0;i<clean.length;i+=2) {
    let a=clean[i], b=clean[i+1]
    if (!b) { b='X' }
    if (a===b) { pairs.push([a,'X']); i-- }
    else pairs.push([a,b])
  }
  const enc = mode.value==='encrypt'?1:-1
  let result=''
  for (const [a,b] of pairs) {
    const [r1,c1]=findPos(m,a), [r2,c2]=findPos(m,b)
    if (r1===r2) { result+=m[r1*5+((c1+enc+5)%5)]; result+=m[r2*5+((c2+enc+5)%5)] }
    else if (c1===c2) { result+=m[((r1+enc+5)%5)*5+c1]; result+=m[((r2+enc+5)%5)*5+c2] }
    else { result+=m[r1*5+c2]; result+=m[r2*5+c1] }
  }
  return result.match(/.{2}/g)?.join(' ') || result
})
</script>
<style scoped>
.pf{display:flex;flex-direction:column;gap:12px}
.pf__field{display:flex;flex-direction:column;gap:4px}
.pf__field label{font-size:12px;color:var(--text-tertiary)}
.pf__field input,.pf__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.pf__actions{display:flex;gap:8px}
.pf__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.pf__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.pf__matrix{display:grid;grid-template-columns:repeat(5,1fr);gap:4px;max-width:200px}
.pf__cell{display:flex;align-items:center;justify-content:center;padding:8px;border:1px solid var(--border-color);border-radius:4px;background:var(--bg-surface);font-family:monospace;font-weight:700;color:var(--text-primary)}
.pf__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:14px;font-weight:600;color:var(--color-primary);min-height:50px;word-break:break-all}
</style>
