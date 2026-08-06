<template>
  <h-single-layout>
    <div class="salsa">
      <div class="salsa__field"><label>密钥 (256位/32字节 Hex)</label><input v-model="keyHex" placeholder="32 Hex 字符" /></div>
      <div class="salsa__field"><label>Nonce (64位/8字节 Hex)</label><input v-model="nonceHex" placeholder="16 Hex 字符" /></div>
      <div class="salsa__field"><label>输入</label><textarea v-model="input" rows="3" :placeholder="mode==='encrypt'?'明文':'Hex密文'" spellcheck="false"></textarea></div>
      <div class="salsa__actions"><button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">加密</button><button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">解密</button></div>
      <div class="salsa__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f20')
const nonceHex = ref('0301040105090206')
const input = ref('Hello Salsa20!')
const mode = ref('encrypt')

function rotl32(x, n) { return ((x << n) | (x >>> (32 - n))) >>> 0 }
function qr(state, a, b, c, d) {
  state[b] ^= rotl32((state[a] + state[d]) | 0, 7)
  state[c] ^= rotl32((state[b] + state[a]) | 0, 9)
  state[d] ^= rotl32((state[c] + state[b]) | 0, 13)
  state[a] ^= rotl32((state[d] + state[c]) | 0, 18)
}

function salsa20Block(key, nonce, counter) {
  const c = 'expand 32-byte k'
  const constants = [c.charCodeAt(0)|c.charCodeAt(1)<<8|c.charCodeAt(2)<<16|c.charCodeAt(3)<<24, c.charCodeAt(4)|c.charCodeAt(5)<<8|c.charCodeAt(6)<<16|c.charCodeAt(7)<<24, c.charCodeAt(8)|c.charCodeAt(9)<<8|c.charCodeAt(10)<<16|c.charCodeAt(11)<<24, c.charCodeAt(12)|c.charCodeAt(13)<<8|c.charCodeAt(14)<<16|c.charCodeAt(15)<<24]
  const state = new Uint32Array(16)
  state[0]=constants[0]; state[5]=constants[1]; state[10]=constants[2]; state[15]=constants[3]
  for (let i=0;i<4;i++){state[1+i*4+i%1]=key[i]; state[2]=key[1]; state[3]=key[2]; state[4]=key[3]; state[11]=key[4]; state[12]=key[5]; state[13]=key[6]; state[14]=key[7]}
  state[1]=key[0];state[2]=key[1];state[3]=key[2];state[4]=key[3]
  state[11]=key[4];state[12]=key[5];state[13]=key[6];state[14]=key[7]
  state[6]=nonce[0];state[7]=nonce[1]
  state[8]=counter;state[9]=0

  const orig = new Uint32Array(state)
  for (let i=0;i<10;i++){qr(state,0,4,8,12);qr(state,5,9,13,1);qr(state,10,14,2,6);qr(state,15,3,7,11);qr(state,0,1,2,3);qr(state,5,6,7,4);qr(state,10,11,8,9);qr(state,15,12,13,14)}
  for (let i=0;i<16;i++) state[i]=(state[i]+orig[i])>>>0
  return state
}

function hexToU32(hex, count) {
  const u32 = []
  for (let i=0;i<count;i++) u32.push(parseInt(hex.substr(i*8,8)||'00000000',16)>>>0)
  return u32
}

const output = computed(() => {
  try {
    const key = hexToU32(keyHex.value.replace(/\s/g,''), 8)
    const nonce = hexToU32(nonceHex.value.replace(/\s/g,''), 2)
    let data
    if (mode.value === 'encrypt') {
      data = Array.from(new TextEncoder().encode(input.value))
    } else {
      const hex = input.value.replace(/\s/g,'')
      data = []
      for (let i=0;i<hex.length;i+=2) data.push(parseInt(hex.substr(i,2),16))
    }
    const result = []
    for (let off=0; off<data.length; off+=64) {
      const block = salsa20Block(key, nonce, Math.floor(off/64)>>>0)
      for (let i=0;i<64 && off+i<data.length;i++) {
        const wordIdx = Math.floor(i/4), byteIdx = i%4
        result.push(data[off+i] ^ ((block[wordIdx] >>> (byteIdx*8)) & 0xFF))
      }
    }
    if (mode.value === 'encrypt') return result.map(b=>b.toString(16).padStart(2,'0')).join('')
    return new TextDecoder().decode(new Uint8Array(result))
  } catch(e) { return '❌ '+e.message }
})
</script>
<style scoped>
.salsa{display:flex;flex-direction:column;gap:12px}.salsa__field{display:flex;flex-direction:column;gap:4px}.salsa__field label{font-size:12px;color:var(--text-tertiary)}.salsa__field input,.salsa__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none;resize:vertical}.salsa__actions{display:flex;gap:8px}.salsa__actions button{padding:8px 16px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-secondary);cursor:pointer}.salsa__actions button.active{background:var(--color-primary);color:white;border-color:var(--color-primary)}.salsa__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:13px;min-height:60px;word-break:break-all}
</style>
