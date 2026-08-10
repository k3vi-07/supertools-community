<template>
  <h-single-layout>
    <div class="idea-tool">
      <div class="idea-tool__field"><label>密钥 (Hex, 32 字符 = 128 位)</label><input v-model="keyHex" placeholder="0123456789abcdef0123456789abcdef" spellcheck="false" /></div>
      <div class="idea-tool__field"><label>输入</label><textarea v-model="input" rows="3" :placeholder="mode==='encrypt'?'明文 Hex (16 字符 = 64 位)':'密文 Hex (16 字符)'" spellcheck="false"></textarea></div>
      <div class="idea-tool__actions">
        <button :class="{active:mode==='encrypt'}" @click="mode='encrypt'">IDEA 加密</button>
        <button :class="{active:mode==='decrypt'}" @click="mode='decrypt'">IDEA 解密</button>
      </div>
      <div class="idea-tool__output selectable">{{ output }}</div>
      <div class="idea-tool__hint">IDEA — 64 位分组，128 位密钥，8.5 轮。基于模 2^16+1 乘法、模 2^16 加法、异或三种运算。Lai & Massey 1991</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const keyHex = ref('00010002000300040005000600070008')
const input = ref('0000000100020003')
const mode = ref('encrypt')

// IDEA uses three group operations on 16-bit blocks:
//   * bitwise XOR
//   * addition modulo 2^16
//   * multiplication modulo 2^16 + 1 (0 represents 2^16)
const MOD = 0x10001 // 65537

// Multiplication modulo 2^16+1. 0x0000 is treated as 2^16.
function mul(a, b) {
  a = a & 0xffff
  b = b & 0xffff
  if (a === 0) a = 0x10000
  if (b === 0) b = 0x10000
  const r = (a * b) % MOD
  return r & 0xffff
}

// Additive inverse mod 2^16
function addInv(a) {
  return (0x10000 - (a & 0xffff)) & 0xffff
}

// Multiplicative inverse mod 2^16+1 using extended Euclidean
function mulInv(a) {
  a = a & 0xffff
  if (a <= 1) return a // 0->0, 1->1
  // Extended Euclidean on (a, MOD)
  let y = 1, x = 0, b = MOD, aa = a
  while (aa > 1) {
    const q = Math.floor(b / aa)
    const t = b % aa
    b = aa
    aa = t
    const xt = (x - q * y) % MOD
    x = y
    y = xt
  }
  if (y < 0) y += MOD
  return y & 0xffff
}

// Key schedule: produce 52 16-bit subkeys from 128-bit key.
// IDEA generates keys by rotating the 128-bit register 25 bits each round.
function expandKey(keyBytes) {
  // Store key as a bit array (128 bits) for rotation
  let bits = []
  for (const b of keyBytes) {
    for (let i = 7; i >= 0; i--) bits.push((b >> i) & 1)
  }
  const subkeys = []
  // Need 52 subkeys -> 52*16 = 832 bits -> 7 rounds of 128-bit register
  let needed = 52
  while (subkeys.length < needed) {
    // Extract subkeys from current 128-bit block
    for (let i = 0; i < 8 && subkeys.length < needed; i++) {
      let v = 0
      for (let j = 0; j < 16; j++) v = (v << 1) | bits[i * 16 + j]
      subkeys.push(v & 0xffff)
    }
    if (subkeys.length < needed) {
      // Rotate bits left by 25
      const rotated = []
      for (let i = 0; i < 128; i++) rotated.push(bits[(i + 25) % 128])
      bits = rotated
    }
  }
  return subkeys
}

// Compute decryption subkeys from encryption subkeys (Schneier, Applied Cryptography).
// Decryption round 1 mirrors the output transform (no middle swap),
// rounds 2..8 mirror encryption rounds 8..2 (with the two middle keys swapped),
// and round 9 (output transform of decrypt) mirrors encryption round 1 (no swap, no MA).
function invertSubkeys(K) {
  const DK = new Array(52).fill(0)
  // Decryption round 1 — mirrors output transform, MA keys from encryption round 8
  DK[0] = mulInv(K[48]); DK[1] = addInv(K[49]); DK[2] = addInv(K[50]); DK[3] = mulInv(K[51])
  DK[4] = K[46]; DK[5] = K[47]
  // Decryption rounds 2..8 — mirror encryption rounds 8..2, with middle swap
  for (let r = 2; r <= 8; r++) {
    const e = 10 - r                    // encryption round mirrored (8..2)
    const eoff = 6 * (e - 1)
    const doff = 6 * (r - 1)
    DK[doff + 0] = mulInv(K[eoff + 0])
    DK[doff + 1] = addInv(K[eoff + 2])  // middle swap
    DK[doff + 2] = addInv(K[eoff + 1])
    DK[doff + 3] = mulInv(K[eoff + 3])
    // MA keys come from the encryption round preceding the mirrored one
    const mae = 6 * (e - 1 - 1)
    DK[doff + 4] = K[mae + 4]
    DK[doff + 5] = K[mae + 5]
  }
  // Decryption output transform — mirrors encryption round 1, no MA keys
  DK[48] = mulInv(K[0]); DK[49] = addInv(K[1]); DK[50] = addInv(K[2]); DK[51] = mulInv(K[3])
  return DK
}

// IDEA cipher (Lai–Massey scheme) on four 16-bit words. keys = 52 subkeys.
function ideaCipher(x1, x2, x3, x4, keys) {
  for (let r = 0; r < 8; r++) {
    const off = r * 6
    // Initial key mixing on the 4 words
    const s1 = mul(x1, keys[off + 0])
    const s2 = (x2 + keys[off + 1]) & 0xffff
    const s3 = (x3 + keys[off + 2]) & 0xffff
    const s4 = mul(x4, keys[off + 3])
    // MA (Multiplication-Addition) structure
    const t1 = s1 ^ s3
    const t2 = s2 ^ s4
    const m1 = mul(t1, keys[off + 4])
    const m2 = mul((m1 + t2) & 0xffff, keys[off + 5])
    const out2 = (m1 + m2) & 0xffff
    // Route MA outputs back: s1 & s3 get m2, s2 & s4 get (m1+m2)
    x1 = s1 ^ m2
    x2 = s2 ^ out2
    x3 = s3 ^ m2
    x4 = s4 ^ out2
    // Swap the middle two words between rounds (not after the last round)
    if (r < 7) {
      const tmp = x2; x2 = x3; x3 = tmp
    }
  }
  // Output transformation: mul/add/mul/add on (x1,x2,x3,x4)
  return [
    mul(x1, keys[48]),
    (x2 + keys[49]) & 0xffff,
    (x3 + keys[50]) & 0xffff,
    mul(x4, keys[51])
  ]
}

function hexToBytes(hex) {
  const out = []
  for (let i = 0; i < hex.length; i += 2) out.push(parseInt(hex.substr(i, 2), 16))
  return out
}

const output = computed(() => {
  try {
    const kc = keyHex.value.replace(/\s/g, '').toLowerCase()
    if (kc.length !== 32) return '❌ 密钥必须为 32 Hex 字符 (128 位)'
    const keyBytes = hexToBytes(kc)
    if (keyBytes.length !== 16) return '❌ 密钥必须为 16 字节'
    const ic = input.value.replace(/\s/g, '').toLowerCase()
    if (ic.length !== 16) return '❌ 数据必须为 16 Hex 字符 (64 位)'
    const data = hexToBytes(ic)
    const x0 = (data[0] << 8) | data[1]
    const x1 = (data[2] << 8) | data[3]
    const x2 = (data[4] << 8) | data[5]
    const x3 = (data[6] << 8) | data[7]
    const encKeys = expandKey(keyBytes)
    const keys = mode.value === 'encrypt' ? encKeys : invertSubkeys(encKeys)
    const [y0, y1, y2, y3] = ideaCipher(x0, x1, x2, x3, keys)
    const out = [
      (y0 >> 8) & 0xff, y0 & 0xff,
      (y1 >> 8) & 0xff, y1 & 0xff,
      (y2 >> 8) & 0xff, y2 & 0xff,
      (y3 >> 8) & 0xff, y3 & 0xff
    ]
    return out.map(b => b.toString(16).padStart(2, '0')).join('')
  } catch (e) {
    return '❌ ' + e.message
  }
})
</script>

<style scoped>
.idea-tool { display: flex; flex-direction: column; gap: 12px; }
.idea-tool__field { display: flex; flex-direction: column; gap: 4px; }
.idea-tool__field label { font-size: 12px; color: var(--text-tertiary); }
.idea-tool__field input, .idea-tool__field textarea { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; outline: none; resize: vertical; }
.idea-tool__actions { display: flex; gap: 8px; }
.idea-tool__actions button { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.idea-tool__actions button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.idea-tool__output { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-family: monospace; font-size: 13px; min-height: 60px; word-break: break-all; }
.idea-tool__hint { font-size: 12px; color: var(--text-tertiary); }
</style>
