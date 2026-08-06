<template>
  <h-single-layout>
    <div class="rabbit">
      <div class="rabbit__field"><label>密钥 (128位/16字节 Hex)</label><input v-model="keyHex" placeholder="32 Hex 字符" /></div>
      <div class="rabbit__field"><label>输入</label><textarea v-model="input" rows="3" :placeholder="mode==='encrypt'?'明文':'Hex密文'" spellcheck="false"></textarea></div>
      <div class="rabbit__actions"><button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">加密</button><button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button></div>
      <div class="rabbit__output selectable">{{ output }}</div>
      <div class="rabbit__hint">Rabbit — 高速流密码，RFC 6631，128 位密钥</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('000102030405060708090a0b0c0d0e0f')
const input = ref('Hello Rabbit!')
const mode = ref('encrypt')

const rotl32 = (x,n) => ((x<<n)|(x>>>(32-n)))>>>0

function rabbitInit(key) {
  const x = new Uint32Array(8), c = new Uint32Array(8)
  const k = new Uint32Array(4)
  for (let i=0;i<4;i++) k[i]=(key[i*4]|key[i*4+1]<<8|key[i*4+2]<<16|key[i*4+3]<<24)>>>0
  for (let i=0;i<8;i+=2) { x[i]=k[(i/2)%4]; x[i+1]=(k[(i/2)%4]>>>16)|(k[((i/2)+1)%4]<<16) }
  for (let i=0;i<8;i+=2) { c[i]=k[(i/2+2)%4]; c[i+1]=(k[(i/2+2)%4]<<16)|(k[(i/2+3)%4]>>>16) }
  c[4]^=x[4]; c[5]^=x[5]; c[6]^=x[6]; c[7]^=x[7]
  return { x, c, carry:0 }
}

function nextSubstep(s) {
  const { x, c, carry } = s
  const g = new Uint32Array(8)
  for (let i=0;i<8;i++) {
    let sum = c[i] + 0xFFFFFFFF + carry
    g[i] = sum & 0xFFFFFFFF
  }
  return { x, c: g }
}

function rabbitNext(s) {
  // Counter update
  const a = [0x4D34D34D, 0xD34D34D3]
  const newC = new Uint32Array(8)
  let ci = s.carry
  for (let i=0;i<8;i++) {
    const temp = (s.c[i] + a[i%2] + ci) >>> 0
    ci = (temp < s.c[i] || (temp === s.c[i] && ci === 1)) ? 1 : 0
    newC[i] = temp
  }
  s.c = newC; s.carry = ci

  // Next-state function
  const g = new Uint32Array(8)
  for (let i=0;i<8;i++) {
    const sum = (s.x[i] + s.c[i]) | 0
    g[i] = Math.imul(sum*sum, 1) >>> 0 ^ (sum >>> 0)
  }
  const newX = new Uint32Array(8)
  newX[0] = (g[0] + rotl32(g[7],16) + rotl32(g[6],16)) >>> 0
  newX[1] = (g[1] + rotl32(g[0],8) + g[6]) >>> 0
  newX[2] = (g[2] + rotl32(g[1],16) + rotl32(g[7],16)) >>> 0
  newX[3] = (g[3] + rotl32(g[2],8) + g[0]) >>> 0
  newX[4] = (g[4] + rotl32(g[3],16) + rotl32(g[2],16)) >>> 0
  newX[5] = (g[5] + rotl32(g[4],8) + g[3]) >>> 0
  newX[6] = (g[6] + rotl32(g[5],16) + rotl32(g[4],16)) >>> 0
  newX[7] = (g[7] + rotl32(g[6],8) + g[5]) >>> 0
  s.x = newX

  // Extract output
  return [s.x[0] ^ (s.x[5]>>>16), s.x[2] ^ (s.x[7]<<16 || 0), s.x[4] ^ (s.x[1]>>>16), s.x[6] ^ (s.x[3]>>>16)]
}

function hexToBytes(hex) {
  const bytes = []
  for (let i=0;i<hex.length;i+=2) bytes.push(parseInt(hex.substr(i,2),16))
  return bytes
}

const output = computed(() => {
  try {
    const keyBytes = hexToBytes(keyHex.value.replace(/\s/g,'').padEnd(32,'0'))
    const state = rabbitInit(keyBytes)
    // Run setup iterations
    for (let i=0;i<4;i++) rabbitNext(state)

    let data
    if (mode.value === 'encrypt') data = Array.from(new TextEncoder().encode(input.value))
    else data = hexToBytes(input.value.replace(/\s/g,''))

    const result = []
    for (let off=0; off<data.length; off+=16) {
      const keystream = rabbitNext(state)
      for (let i=0;i<16 && off+i<data.length;i++) {
        const wordIdx = Math.floor(i/4), byteIdx = i%4
        result.push(data[off+i] ^ ((keystream[wordIdx] >>> (byteIdx*8)) & 0xFF))
      }
    }
    if (mode.value === 'encrypt') return result.map(b=>b.toString(16).padStart(2,'0')).join('')
    return new TextDecoder().decode(new Uint8Array(result))
  } catch(e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.rabbit{display:flex;flex-direction:column;gap:12px}
.rabbit__field{display:flex;flex-direction:column;gap:4px}
.rabbit__field label{font-size:12px;color:var(--text-tertiary)}
.rabbit__field input,.rabbit__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}
.rabbit__actions{display:flex;gap:8px}
.rabbit__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}
.rabbit__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}
.rabbit__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
.rabbit__hint{font-size:12px;color:var(--text-tertiary)}
</style>
