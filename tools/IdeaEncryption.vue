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

// Compute decryption subkeys from encryption subkeys
function invertSubkeys(encKeys) {
  const dec = new Array(52)
  for (let r = 0; r < 9; r++) {
    const encOff = r * 6
    const decOff = (8 - r) * 6
    if (r === 0) {
      dec[decOff + 0] = mulInv(encKeys[encOff + 0])
      dec[decOff + 1] = addInv(encKeys[encOff + 1])
      dec[decOff + 2] = addInv(encKeys[encOff + 2])
      dec[decOff + 3] = mulInv(encKeys[encOff + 3])
    } else {
      // For rounds 1..7 the two middle keys are swapped
      dec[decOff + 0] = mulInv(encKeys[encOff + 0])
      dec[decOff + 1] = addInv(encKeys[encOff + 2])
      dec[decOff + 2] = addInv(encKeys[encOff + 1])
      dec[decOff + 3] = mulInv(encKeys[encOff + 3])
    }
    if (r < 8) {
      // The half-round subkeys
      dec[decOff + 4] = encKeys[encOff - 2 + 6] // maps to next round's MA keys, simplified
      dec[decOff + 5] = encKeys[encOff - 1 + 6]
    }
  }
  // Handle the MA subkeys (positions 4,5 of each round) properly.
  // Rebuild cleanly: round r (0..7) uses encKeys[r*6+4], encKeys[r*6+5].
  // Decryption round (8-r) should use the SAME MA keys as encryption round r.
  for (let r = 0; r < 8; r++) {
    const decOff = (8 - r) * 6
    dec[decOff + 4] = encKeys[r * 6 + 4]
    dec[decOff + 5] = encKeys[r * 6 + 5]
  }
  return dec
}

// IDEA cipher on 4 16-bit words. keys = 52 subkeys.
function ideaCipher(x0, x1, x2, x3, keys) {
  for (let r = 0; r < 8; r++) {
    const off = r * 6
    // Apply key mixing
    x0 = mul(x0, keys[off + 0])
    x1 = (x1 + keys[off + 1]) & 0xffff
    x2 = (x2 + keys[off + 2]) & 0xffff
    x3 = mul(x3, keys[off + 3])
    // MA structure
    const t1 = x0 ^ x2
    const t2 = x1 ^ x3
    const m1 = mul(t1, keys[off + 4])
    const m2 = mul(m1, t2)
    const m3 = mul(m1 + m2, keys[off + 5])
    const t3 = (m1 + m3) & 0xffff
    // Cross swap
    x0 = x0 ^ m2
    x2 = x2 ^ m2
    x1 = x1 ^ t3
    x3 = x3 ^ t3
    // Swap x1 and x2 (except after last round)
    if (r < 7) {
      const tmp = x1; x1 = x2; x2 = tmp
    }
  }
  // Final output transformation (keys[48..51])
  const y0 = mul(x0, keys[48])
  const y1 = (x2 + keys[49]) & 0xffff // note swap of x1/x2 already done
  const y2 = (x1 + keys[50]) & 0xffff
  const y3 = mul(x3, keys[51])
  return [y0, y1, y2, y3]
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
