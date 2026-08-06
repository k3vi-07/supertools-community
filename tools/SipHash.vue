<template>
  <h-single-layout>
    <div class="siphash">
      <div class="siphash__field"><label>输入</label><textarea v-model="input" rows="3" spellcheck="false"></textarea></div>
      <div class="siphash__field"><label>密钥 (16字节 Hex)</label><input v-model="keyHex" placeholder="000102030405060708090a0b0c0d0e0f" /></div>
      <div class="siphash__field"><label>压缩轮数</label>
        <div class="siphash__radios">
          <label><input type="radio" value="2-4" v-model="rounds" /> SipHash-2-4 (标准)</label>
          <label><input type="radio" value="1-3" v-model="rounds" /> SipHash-1-3 (快速)</label>
        </div>
      </div>
      <div class="siphash__output selectable">{{ output }}</div>
      <div class="siphash__hint">SipHash — 高性能 PRF，广泛用于哈希表防碰撞攻击保护</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const input = ref('Hello SipHash!')
const keyHex = ref('000102030405060708090a0b0c0d0e0f')
const rounds = ref('2-4')

function rotl(x, b) { return ((x << b) | (x >>> (64n - b))) }
// 用 BigInt 实现 64 位运算
function sipHash64(msg, keyBytes, cRounds, dRounds) {
  const k0 = BigInt('0x' + (keyBytes.slice(0,8).map(b=>b.toString(16).padStart(2,'0')).join('') || '0'))
  const k1 = BigInt('0x' + (keyBytes.slice(8,16).map(b=>b.toString(16).padStart(2,'0')).join('') || '0'))
  const v0 = 0x736f6d6570736575n ^ k0
  const v1 = 0x646f72616e646f6dn ^ k1
  const v2 = 0x6c7967656e657261n ^ k0
  const v3 = 0x7465646279746573n ^ k1

  const sipRound = (s) => {
    s[0] = (s[0] + s[1]) & 0xFFFFFFFFFFFFFFFFn; s[1] = rotl(s[1], 13n); s[1] ^= s[0]; s[0] = rotl(s[0], 32n)
    s[2] = (s[2] + s[3]) & 0xFFFFFFFFFFFFFFFFn; s[3] = rotl(s[3], 16n); s[3] ^= s[2]
    s[0] = (s[0] + s[3]) & 0xFFFFFFFFFFFFFFFFn; s[3] = rotl(s[3], 21n); s[3] ^= s[0]
    s[2] = (s[2] + s[1]) & 0xFFFFFFFFFFFFFFFFn; s[1] = rotl(s[1], 17n); s[1] ^= s[2]; s[2] = rotl(s[2], 32n)
  }

  const state = [v0, v1, v2, v3]
  const len = msg.length
  const lastBlockIdx = len & ~7

  for (let i = 0; i < lastBlockIdx; i += 8) {
    let m = 0n
    for (let j = 7; j >= 0; j--) m = (m << 8n) | BigInt(msg[i + j])
    state[3] ^= m
    for (let r = 0; r < cRounds; r++) sipRound(state)
    state[0] ^= m
  }

  let b = BigInt(len) << 56n
  for (let j = 0; j < (len & 7); j++) b |= BigInt(msg[lastBlockIdx + j]) << BigInt(j * 8)
  state[3] ^= b
  for (let r = 0; r < cRounds; r++) sipRound(state)
  state[0] ^= b
  state[2] ^= 0xFFn

  for (let r = 0; r < dRounds; r++) sipRound(state)
  return (state[0] ^ state[1] ^ state[2] ^ state[3]).toString(16).padStart(16, '0')
}

const output = computed(() => {
  try {
    const msg = new TextEncoder().encode(input.value)
    let keyBytes = []
    const hex = keyHex.value.replace(/\s/g, '')
    for (let i = 0; i < 16; i++) keyBytes.push(i*2 < hex.length ? parseInt(hex.substr(i*2, 2) || '0', 16) : 0)
    const [c, d] = rounds.value.split('-').map(Number)
    return sipHash64(msg, keyBytes, c, d).toUpperCase()
  } catch(e) { return '❌ ' + e.message }
})
</script>
<style scoped>
.siphash { display:flex; flex-direction:column; gap:12px; }
.siphash__field { display:flex; flex-direction:column; gap:4px; }
.siphash__field label { font-size:12px; color:var(--text-tertiary); }
.siphash__field input, .siphash__field textarea { padding:8px 12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); color:var(--text-primary); font-family:monospace; outline:none; resize:vertical; }
.siphash__radios { display:flex; gap:16px; }
.siphash__radios label { font-size:13px; color:var(--text-secondary); display:flex; align-items:center; gap:4px; cursor:pointer; }
.siphash__output { padding:12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-base); font-family:monospace; font-size:16px; font-weight:700; color:var(--color-primary); min-height:50px; word-break:break-all; }
.siphash__hint { font-size:12px; color:var(--text-tertiary); }
</style>
