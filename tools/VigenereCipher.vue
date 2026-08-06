<template>
  <h-single-layout>
    <div class="vig">
      <div class="vig__tabs">
        <button v-for="c in ciphers" :key="c.id" :class="{active:cipher===c.id}" @click="cipher=c.id">{{ c.name }}</button>
      </div>
      <div v-if="cipher==='vigenere'" class="vig__field"><label>密钥</label><input v-model="vigenereKey" placeholder="字母密钥" /></div>
      <div v-if="cipher==='caesar'" class="vig__field"><label>位移</label><input type="number" v-model.number="caesarShift" min="1" max="25" /></div>
      <div v-if="cipher==='affine'" class="vig__cols">
        <div class="vig__field"><label>a (与 26 互质)</label><input type="number" v-model.number="affineA" /></div>
        <div class="vig__field"><label>b</label><input type="number" v-model.number="affineB" /></div>
      </div>
      <div class="vig__field"><label>输入</label><textarea v-model="input" rows="4" spellcheck="false"></textarea></div>
      <div class="vig__actions"><button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">加密</button><button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button></div>
      <div class="vig__output selectable">{{ output }}</div>
      <button class="vig__copy" @click="copy">复制结果</button>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const ciphers = [
  {id:'caesar',name:'Caesar 凯撒'}, {id:'rot13',name:'ROT13'},
  {id:'vigenere',name:'Vigenère'}, {id:'atbash',name:'Atbash'},
  {id:'affine',name:'Affine 仿射'}, {id:'railfence',name:'Rail Fence 栅栏'},
]
const cipher = ref('caesar')
const caesarShift = ref(3)
const vigenereKey = ref('KEY')
const affineA = ref(5)
const affineB = ref(8)
const railRows = ref(3)
const input = ref('Hello World')
const mode = ref('encrypt')

const output = computed(() => {
  const s = input.value
  const enc = mode.value === 'encrypt' ? 1 : -1
  switch(cipher.value) {
    case 'caesar': return s.replace(/[a-z]/gi, c => { const b=c<='Z'?65:97; return String.fromCharCode((c.charCodeAt(0)-b+enc*caesarShift+26)%26+b) })
    case 'rot13': return s.replace(/[a-z]/gi, c => { const b=c<='Z'?65:97; return String.fromCharCode((c.charCodeAt(0)-b+13)%26+b) })
    case 'vigenere': {
      const key = vigenereKey.value.toUpperCase().replace(/[^A-Z]/g,'') || 'A'
      let ki = 0
      return s.replace(/[a-z]/gi, c => { const b=c<='Z'?65:97; const shift=(key.charCodeAt(ki%key.length)-65)*enc; ki++; return String.fromCharCode((c.charCodeAt(0)-b+shift+26)%26+b) })
    }
    case 'atbash': return s.replace(/[a-z]/gi, c => { const b=c<='Z'?90:122; return String.fromCharCode(b-(c.charCodeAt(0)-(c<='Z'?65:97))) })
    case 'affine': {
      const a = affineA.value, b = affineB.value
      if (enc === 1) return s.replace(/[a-z]/gi, c => { const base=c<='Z'?65:97; return String.fromCharCode((a*(c.charCodeAt(0)-base)+b)%26+base) })
      // 解密需要 a 的模反
      let aInv = 1
      for (let i=1;i<26;i++) if ((a*i)%26===1) { aInv=i; break }
      return s.replace(/[a-z]/gi, c => { const base=c<='Z'?65:97; return String.fromCharCode((aInv*((c.charCodeAt(0)-base)-b+26*10))%26+base) })
    }
    case 'railfence': {
      const rows = railRows.value
      if (enc === 1) {
        const rails = Array.from({length:rows},()=>[])
        let r=0,dir=1
        for (const ch of s) { rails[r].push(ch); r+=dir; if(r===0||r===rows-1) dir=-dir }
        return rails.flat().join('')
      } else {
        // 解密栅栏
        const pattern=[],rails=Array.from({length:rows},()=>[])
        let r=0,dir=1
        for(let i=0;i<s.length;i++){pattern.push(r);rails[r].push(i);r+=dir;if(r===0||r===rows-1)dir=-dir}
        const railLens=rails.map(x=>x.length)
        let idx=0
        const rails2=Array.from({length:rows},()=>[])
        for(let i=0;i<rows;i++){rails2[i]=s.substr(idx,railLens[i]).split('');idx+=railLens[i]}
        let result='',ri=Array(rows).fill(0)
        for(const pr of pattern){result+=rails2[pr][ri[pr]++]}
        return result
      }
    }
    default: return s
  }
})
function copy(){window.$he3?.copyText(output.value);window.$he3?.message.success('已复制')}
</script>
<style scoped>
.vig{display:flex;flex-direction:column;gap:12px}
.vig__tabs{display:flex;gap:4px;flex-wrap:wrap}
.vig__tabs button{padding:4px 10px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer;font-size:12px}
.vig__tabs button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.vig__cols{display:flex;gap:8px}
.vig__cols .vig__field{flex:1}
.vig__field{display:flex;flex-direction:column;gap:4px}
.vig__field label{font-size:12px;color:var(--text-tertiary)}
.vig__field input,.vig__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.vig__actions{display:flex;gap:8px}
.vig__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.vig__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.vig__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.vig__copy{padding:8px 16px;border:none;border-radius:8px;background:var(--color-primary);color:white;cursor:pointer;align-self:flex-start}
</style>
