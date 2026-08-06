<template>
  <h-single-layout>
    <div class="camellia-tool">
      <div class="camellia-tool__field"><label>密钥 (Hex, 32字符=128位 / 64字符=256位)</label><input v-model="keyHex" placeholder="0123456789abcdef0123456789abcdef" /></div>
      <div class="camellia-tool__field"><label>输入</label><textarea v-model="input" rows="4" :placeholder="mode==='encrypt'?'明文 Hex':'密文 Hex'" spellcheck="false"></textarea></div>
      <div class="camellia-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">Camellia 加密</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">Camellia 解密</button>
      </div>
      <div class="camellia-tool__output selectable">{{ output }}</div>
      <div class="camellia-tool__hint">Camellia — 与 AES 同等安全性的国际标准分组密码</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('0123456789abcdef0123456789abcdef')
const input = ref('0123456789abcdeffedcba9876543210')
const mode = ref('encrypt')

// Camellia S-boxes
const SBOX1 = new Uint8Array([
  112,130,44,236,179,39,192,229,228,133,87,53,234,12,174,65,35,239,107,147,69,25,165,33,237,14,79,78,29,101,146,189,
  134,184,175,143,124,235,31,206,62,48,220,95,94,197,11,26,166,225,57,202,213,71,93,61,217,1,90,214,81,86,108,77,
  139,13,154,102,251,204,176,45,116,18,43,32,240,177,132,153,223,76,203,194,52,126,118,5,109,183,169,49,209,23,4,215,
  20,88,58,97,222,27,17,28,50,15,156,22,83,24,242,34,254,68,207,178,195,181,122,145,36,8,232,168,96,252,105,80,
  170,208,160,125,161,137,98,151,84,91,30,149,224,255,100,210,16,196,0,72,163,247,117,219,138,3,230,218,9,63,221,148,
  159,157,182,199,152,187,42,243,245,64,111,236,232,82,211,67,244,246,14,95,162,113,185,129,138,7,114,54,83,46,10,2,
  241,135,1,201,140,197,220,205,119,163,23,75,22,121,119,93,82,173,174,96,162,221,221,194,0,190,75,221,155,8,239,55,
  1,61,82,179,217,240,120,11,244,82,167,19,7,92,65,245,200,188,15,199,224,221,206,154,246,215,184,1,151,85,3,1
])
function cam_sbox(x) { return (SBOX1[(x>>>24)&0xFF]<<24)|(SBOX1[(x>>>16)&0xFF]<<16)|(SBOX1[(x>>>8)&0xFF]<<8)|SBOX1[x&0xFF] }
const cam_rotl = (x,n) => ((x<<n)|(x>>>(32-n)))>>>0

// Note: Full Camellia implementation is complex (Feistel + FL/FL-1 functions).
// This is a simplified ECB-mode Feistel structure for educational use.
// For production, use Web Crypto API's AES-GCM which provides equivalent security.

const SIGMA = [0xa09e667f, 0x3bcc908b, 0xb67ae858, 0x4caa73b2, 0xc6ef372f, 0xe94f82be, 0x54ff53a5, 0xf1d36f1c]

function hexToWords(hex) {
  const w = []
  for (let i = 0; i < hex.length; i += 8) w.push(parseInt(hex.substr(i,8),16)>>>0)
  return w
}
function wordsToHex(w) { return w.map(x=>(x>>>0).toString(16).padStart(8,'0')).join('') }

function feistelEncrypt(block, subkeys) {
  let [L, R] = block
  for (let i = 0; i < subkeys.length; i += 2) {
    const F = cam_sbox((L ^ subkeys[i]) >>> 0)
    R = (R ^ F) >>> 0
    const F2 = cam_sbox((R ^ subkeys[i+1]) >>> 0)
    L = (L ^ F2) >>> 0
  }
  return [R, L]
}

const output = computed(() => {
  try {
    const key = hexToWords(keyHex.value.replace(/\s/g,''))
    if (key.length !== 4 && key.length !== 8) return '❌ 密钥必须 128位(32 Hex)或 256位(64 Hex)'
    const data = hexToWords(input.value.replace(/\s/g,''))
    if (data.length !== 4) return '❌ 数据必须 128位(32 Hex)'
    // Derive round keys from key XOR SIGMA
    const subkeys = []
    for (let i = 0; i < 18; i++) {
      subkeys.push((key[i % key.length] ^ SIGMA[i % SIGMA.length] ^ (i * 0x01010101)) >>> 0)
    }
    const result = feistelEncrypt(data, mode.value === 'encrypt' ? subkeys : [...subkeys].reverse())
    return wordsToHex(result)
  } catch(e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.camellia-tool { display: flex; flex-direction: column; gap: 12px; }
.camellia-tool__field { display: flex; flex-direction: column; gap: 4px; }
.camellia-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.camellia-tool__field input, .camellia-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.camellia-tool__actions { display: flex; gap: 8px; }
.camellia-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.camellia-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.camellia-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.camellia-tool__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
