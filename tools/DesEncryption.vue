<template>
  <h-single-layout>
    <div class="des-enc">
      <div class="des-enc__mode">
        <label>模式</label>
        <select v-model="mode">
          <option value="des-ecb">DES ECB</option>
          <option value="des-cbc">DES CBC</option>
          <option value="3des-ecb">3DES ECB</option>
          <option value="3des-cbc">3DES CBC</option>
        </select>
      </div>

      <div class="des-enc__fields">
        <div class="des-enc__field">
          <label>密钥 (Hex)</label>
          <input v-model="keyHex" class="des-enc__input" spellcheck="false" />
          <small>{{ keyLengthHint }}</small>
        </div>
        <div class="des-enc__field" v-if="mode.includes('cbc')">
          <label>IV (Hex)</label>
          <input v-model="ivHex" class="des-enc__input" spellcheck="false" />
          <small>8 字节 (16 hex)</small>
        </div>
      </div>

      <div class="des-enc__io">
        <textarea v-model="input" class="des-enc__text selectable" placeholder="输入明文..." spellcheck="false"></textarea>
      </div>

      <div class="des-enc__actions">
        <button class="des-enc__btn des-enc__btn--primary" @click="doEncrypt">加密</button>
        <button class="des-enc__btn" @click="doDecrypt">解密</button>
      </div>

      <div class="des-enc__output-area">
        <label>结果 (Hex)</label>
        <div class="des-enc__output selectable">{{ output || '(点击加密/解密)' }}</div>
        <button v-if="output" class="des-enc__copy" @click="copyOutput">复制</button>
      </div>

      <div v-if="error" class="des-enc__error">{{ error }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const mode = ref('des-ecb')
const keyHex = ref('133457799bbcdff1')
const ivHex = ref('0123456789abcdef')
const input = ref('Hello DES!')
const output = ref('')
const error = ref('')

const keyLengthHint = computed(() => {
  if (mode.value.startsWith('3des')) return '24 字节 (48 hex)'
  return '8 字节 (16 hex)'
})

// ===== DES 纯 JS 实现 =====
const IP = [58,50,42,34,26,18,10,2,60,52,44,36,28,20,12,4,62,54,46,38,30,22,14,6,64,56,48,40,32,24,16,8,57,49,41,33,25,17,9,1,59,51,43,35,27,19,11,3,61,53,45,37,29,21,13,5,63,55,47,39,31,23,15,7]
const IP_INV = [40,8,48,16,56,24,64,32,39,7,47,15,55,23,63,31,38,6,46,14,54,22,62,30,37,5,45,13,53,21,61,29,36,4,44,12,52,20,60,28,35,3,43,11,51,19,59,27,34,2,42,10,50,18,58,26,33,1,41,9,49,17,57,25]
const E = [32,1,2,3,4,5,4,5,6,7,8,9,8,9,10,11,12,13,12,13,14,15,16,17,16,17,18,19,20,21,20,21,22,23,24,25,24,25,26,27,28,29,28,29,30,31,32,1]
const P = [16,7,20,21,29,12,28,17,1,15,23,26,5,18,31,10,2,8,24,14,32,27,3,9,19,13,30,6,22,11,4,25]
const PC1 = [57,49,41,33,25,17,9,1,58,50,42,34,26,18,10,2,59,51,43,35,27,19,11,3,60,52,44,36,63,55,47,39,31,23,15,7,62,54,46,38,30,22,14,6,61,53,45,37,29,21,13,5,28,20,12,4]
const PC2 = [14,17,11,24,1,5,3,28,15,6,21,10,23,19,12,4,26,8,16,7,27,20,13,2,41,52,31,37,47,55,30,40,51,45,33,48,44,49,39,56,34,53,46,42,50,36,29,32]
const SHIFTS = [1,1,2,2,2,2,2,2,1,2,2,2,2,2,2,1]

const S_BOXES = [
  [[14,4,13,1,2,15,11,8,3,10,6,12,5,9,0,7],[0,15,7,4,14,2,13,1,10,6,12,11,9,5,3,8],[4,1,14,8,13,6,2,11,15,12,9,7,3,10,5,0],[15,12,8,2,4,9,1,7,5,11,3,14,10,0,6,13]],
  [[15,1,8,14,6,11,3,4,9,7,2,13,12,0,5,10],[3,13,4,7,15,2,8,14,12,0,1,10,6,9,11,5],[0,14,7,11,10,4,13,1,5,8,12,6,9,3,2,15],[13,8,10,1,3,15,4,2,11,6,7,12,0,5,14,9]],
  [[10,0,9,14,6,3,15,5,1,13,12,7,11,4,2,8],[13,7,0,9,3,4,6,10,2,8,5,14,12,11,15,1],[13,6,4,9,8,15,3,0,11,1,2,12,5,10,14,7],[1,10,13,0,6,9,8,7,4,15,14,3,11,5,2,12]],
  [[7,13,14,3,0,6,9,10,1,2,8,5,11,12,4,15],[13,8,11,5,6,15,0,3,4,7,2,12,1,10,14,9],[10,6,9,0,12,11,7,13,15,1,3,14,5,2,8,4],[3,15,0,6,10,1,13,8,9,4,5,11,12,7,2,14]],
  [[2,12,4,1,7,10,11,6,8,5,3,15,13,0,14,9],[14,11,2,12,4,7,13,1,5,0,15,10,3,9,8,6],[4,2,1,11,10,13,7,8,15,9,12,5,6,3,0,14],[11,8,12,7,1,14,2,13,6,15,0,9,10,4,5,3]],
  [[12,1,10,15,9,2,6,8,0,13,3,4,14,7,5,11],[10,15,4,2,7,12,9,5,6,1,13,14,0,11,3,8],[9,14,15,5,2,8,12,3,7,0,4,10,1,13,11,6],[4,3,2,12,9,5,15,10,11,14,1,7,6,0,8,13]],
  [[4,11,2,14,15,0,8,13,3,12,9,7,5,10,6,1],[13,0,11,7,4,9,1,10,14,3,5,12,2,15,8,6],[1,4,11,13,12,3,7,14,10,15,6,8,0,5,9,2],[6,11,13,8,1,4,10,7,9,5,0,15,14,2,3,12]],
  [[13,2,8,4,6,15,11,1,10,9,3,14,5,0,12,7],[1,15,13,8,10,3,7,4,12,5,6,11,0,14,9,2],[7,11,4,1,9,12,14,2,0,6,10,13,15,3,5,8],[2,1,14,7,4,10,8,13,15,12,9,0,3,5,6,11]]
]

function permute(bits, table) {
  return table.map(i => bits[i - 1])
}
function leftShift(arr, n) {
  return [...arr.slice(n), ...arr.slice(0, n)]
}
function xor(a, b) {
  return a.map((v, i) => v ^ b[i])
}
function hexToBits(hex) {
  const bits = []
  for (const c of hex) {
    const n = parseInt(c, 16)
    for (let i = 3; i >= 0; i--) bits.push((n >> i) & 1)
  }
  return bits
}
function bitsToHex(bits) {
  let hex = ''
  for (let i = 0; i < bits.length; i += 4) {
    hex += parseInt(bits.slice(i, i + 4).join(''), 2).toString(16)
  }
  return hex
}
function generateSubkeys(key64) {
  const key56 = permute(key64, PC1)
  let c = key56.slice(0, 28)
  let d = key56.slice(28, 56)
  const subkeys = []
  for (let i = 0; i < 16; i++) {
    c = leftShift(c, SHIFTS[i])
    d = leftShift(d, SHIFTS[i])
    subkeys.push(permute([...c, ...d], PC2))
  }
  return subkeys
}
function feistel(r32, subkey) {
  const e = permute(r32, E)
  const x = xor(e, subkey)
  let sout = []
  for (let i = 0; i < 8; i++) {
    const chunk = x.slice(i * 6, (i + 1) * 6)
    const row = parseInt('' + chunk[0] + chunk[5], 2)
    const col = parseInt(chunk.slice(1, 5).join(''), 2)
    const val = S_BOXES[i][row][col]
    for (let j = 3; j >= 0; j--) sout.push((val >> j) & 1)
  }
  return permute(sout, P)
}
function desBlock(block64, subkeys) {
  const ip = permute(block64, IP)
  let l = ip.slice(0, 32)
  let r = ip.slice(32, 64)
  for (let i = 0; i < 16; i++) {
    const newR = xor(l, feistel(r, subkeys[i]))
    l = r
    r = newR
  }
  return permute([...r, ...l], IP_INV)
}
function desEncryptBlock(hexBlock, keyHex) {
  return bitsToHex(desBlock(hexToBits(hexBlock), generateSubkeys(hexToBits(keyHex))))
}
function desDecryptBlock(hexBlock, keyHex) {
  return bitsToHex(desBlock(hexToBits(hexBlock), generateSubkeys(hexToBits(keyHex)).reverse()))
}

function strToHex(str) {
  let hex = ''
  for (let i = 0; i < str.length; i++) {
    hex += str.charCodeAt(i).toString(16).padStart(2, '0')
  }
  return hex
}
function hexToStr(hex) {
  let str = ''
  for (let i = 0; i < hex.length; i += 2) {
    str += String.fromCharCode(parseInt(hex.substr(i, 2), 16))
  }
  return str
}
function pkcs7Pad(hex) {
  const blockSize = 8
  const byteLen = hex.length / 2
  const padLen = blockSize - (byteLen % blockSize)
  let padHex = ''
  for (let i = 0; i < padLen; i++) padHex += padLen.toString(16).padStart(2, '0')
  return hex + padHex
}
function pkcs7Unpad(hex) {
  const padLen = parseInt(hex.substr(hex.length - 2), 16)
  if (padLen < 1 || padLen > 8) return hex
  return hex.slice(0, hex.length - padLen * 2)
}

function getKeyParts(k) {
  // 3DES 需要 3 个 8 字节子密钥
  const k1 = k.substr(0, 16)
  const k2 = k.substr(16, 16) || k1
  const k3 = k.substr(32, 16) || k1
  return [k1, k2, k3]
}

function encryptHex(dataHex, keyHex, ivHex, mode) {
  const padded = pkcs7Pad(dataHex)
  const blocks = padded.match(/.{1,16}/g) || []
  let result = ''
  let prev = ivHex
  const isTriple = mode.startsWith('3des')
  const isCbc = mode.includes('cbc')
  const keys = isTriple ? getKeyParts(keyHex) : [keyHex]
  for (const block of blocks) {
    let x = block
    if (isCbc) x = xorHex(block, prev)
    let enc
    if (isTriple) {
      enc = desEncryptBlock(x, keys[0])
      enc = desDecryptBlock(enc, keys[1])
      enc = desEncryptBlock(enc, keys[2])
    } else {
      enc = desEncryptBlock(x, keys[0])
    }
    result += enc
    if (isCbc) prev = enc
  }
  return result
}
function decryptHex(cipherHex, keyHex, ivHex, mode) {
  const blocks = cipherHex.match(/.{1,16}/g) || []
  let result = ''
  let prev = ivHex
  const isTriple = mode.startsWith('3des')
  const isCbc = mode.includes('cbc')
  const keys = isTriple ? getKeyParts(keyHex) : [keyHex]
  for (const block of blocks) {
    let dec
    if (isTriple) {
      dec = desDecryptBlock(block, keys[2])
      dec = desEncryptBlock(dec, keys[1])
      dec = desDecryptBlock(dec, keys[0])
    } else {
      dec = desDecryptBlock(block, keys[0])
    }
    if (isCbc) dec = xorHex(dec, prev)
    result += dec
    if (isCbc) prev = block
  }
  return pkcs7Unpad(result)
}
function xorHex(a, b) {
  let result = ''
  for (let i = 0; i < a.length; i += 2) {
    result += (parseInt(a.substr(i, 2), 16) ^ parseInt(b.substr(i, 2), 16)).toString(16).padStart(2, '0')
  }
  return result
}

function doEncrypt() {
  error.value = ''
  try {
    const keyBytes = mode.value.startsWith('3des') ? 24 : 8
    if (keyHex.value.length !== keyBytes * 2) throw new Error('密钥长度应为 ' + keyBytes * 2 + ' 个 hex 字符')
    output.value = encryptHex(strToHex(input.value), keyHex.value.toLowerCase(), ivHex.value.toLowerCase(), mode.value)
  } catch (e) {
    error.value = e.message
    output.value = ''
  }
}
function doDecrypt() {
  error.value = ''
  try {
    const dec = decryptHex(input.value.toLowerCase(), keyHex.value.toLowerCase(), ivHex.value.toLowerCase(), mode.value)
    output.value = hexToStr(dec)
  } catch (e) {
    error.value = e.message
    output.value = ''
  }
}
function copyOutput() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.des-enc { display: flex; flex-direction: column; gap: 12px; }
.des-enc__mode { display: flex; align-items: center; gap: 8px; }
.des-enc__mode label { font-size: 13px; color: var(--text-secondary); }
.des-enc__mode select { padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 13px; }
.des-enc__fields { display: flex; gap: 12px; }
.des-enc__field { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.des-enc__field label { font-size: 12px; color: var(--text-tertiary); }
.des-enc__field small { font-size: 11px; color: var(--text-tertiary); }
.des-enc__input { padding: 8px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; }
.des-enc__text { width: 100%; min-height: 80px; padding: 10px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.des-enc__actions { display: flex; gap: 8px; }
.des-enc__btn { padding: 8px 20px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; cursor: pointer; }
.des-enc__btn--primary { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.des-enc__output-area { display: flex; flex-direction: column; gap: 4px; position: relative; }
.des-enc__output-area label { font-size: 12px; color: var(--text-tertiary); }
.des-enc__output { padding: 10px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; min-height: 40px; }
.des-enc__copy { position: absolute; top: 22px; right: 8px; padding: 4px 10px; border: 1px solid var(--color-primary); border-radius: 4px; background: var(--bg-surface); color: var(--color-primary); font-size: 11px; cursor: pointer; }
.des-enc__error { padding: 8px 12px; border-radius: 6px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
</style>
