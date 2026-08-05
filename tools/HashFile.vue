<template>
  <h-single-layout>
    <div class="hash-file">
      <div class="hash-file__dropzone" @dragover.prevent="dragActive = true" @dragleave="dragActive = false" @drop.prevent="onDrop" @click="$refs.fileInput.click()">
        <h-icon icon="mdi:file-upload-outline" :size="36" color="var(--text-tertiary)" />
        <p>{{ fileName || '拖拽文件到这里或点击选择' }}</p>
        <small v-if="fileSize">{{ formatSize(fileSize) }}</small>
        <input ref="fileInput" type="file" hidden @change="onFileSelect" />
      </div>

      <div v-if="results.length > 0" class="hash-file__results">
        <div v-for="r in results" :key="r.algo" class="hash-file__result">
          <span class="hash-file__algo">{{ r.algo }}</span>
          <code class="hash-file__hash selectable">{{ r.hash }}</code>
          <button class="hash-file__copy" @click="copy(r.hash)">复制</button>
        </div>
      </div>

      <div v-if="computing" class="hash-file__progress">
        <span>正在计算...</span>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'

const fileName = ref('')
const fileSize = ref(0)
const results = ref([])
const computing = ref(false)
const dragActive = ref(false)

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / 1048576).toFixed(2) + ' MB'
}

function bufToHex(buf) {
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('')
}

async function hashFile(file) {
  if (!file) return
  fileName.value = file.name
  fileSize.value = file.size
  results.value = []
  computing.value = true

  const buffer = await file.arrayBuffer()
  const algos = ['SHA-1', 'SHA-256', 'SHA-384', 'SHA-512']

  for (const algo of algos) {
    try {
      const hash = await crypto.subtle.digest(algo, buffer)
      results.value.push({ algo, hash: bufToHex(hash) })
    } catch (e) {
      results.value.push({ algo, hash: 'Error: ' + e.message })
    }
  }

  // MD5 不在 Web Crypto 中，尝试用简单的纯 JS 实现（仅对小文件）
  if (file.size < 1024 * 1024) {
    try {
      const md5 = await computeMD5(buffer)
      results.value.push({ algo: 'MD5', hash: md5 })
    } catch {
      // 跳过
    }
  }

  computing.value = false
}

function onDrop(e) {
  dragActive.value = false
  const file = e.dataTransfer.files[0]
  if (file) hashFile(file)
}

function onFileSelect(e) {
  const file = e.target.files[0]
  if (file) hashFile(file)
}

function copy(text) {
  navigator.clipboard?.writeText(text)
}

// 简易 MD5（纯 JS，仅支持小文件）
async function computeMD5(buffer) {
  const bytes = new Uint8Array(buffer)
  // MD5 实现
  const s = unescape(encodeURIComponent(String.fromCharCode.apply(null, bytes)))
  return md5(s)
}

function md5(string) {
  function rotateLeft(lValue, iShiftBits) {
    return (lValue << iShiftBits) | (lValue >>> (32 - iShiftBits))
  }
  function addUnsigned(lX, lY) {
    const lX4 = (lX & 0x40000000)
    const lY4 = (lY & 0x40000000)
    const lX8 = (lX & 0x80000000)
    const lY8 = (lY & 0x80000000)
    const lResult = (lX & 0x3FFFFFFF) + (lY & 0x3FFFFFFF)
    if (lX4 & lY4) return lResult ^ 0x80000000 ^ lX8 ^ lY8
    if (lX4 | lY4) {
      if (lResult & 0x40000000) return lResult ^ 0xC0000000 ^ lX8 ^ lY8
      else return lResult ^ 0x40000000 ^ lX8 ^ lY8
    } else return lResult ^ lX8 ^ lY8
  }
  function F(x, y, z) { return (x & y) | ((~x) & z) }
  function G(x, y, z) { return (x & z) | (y & (~z)) }
  function H(x, y, z) { return (x ^ y ^ z) }
  function I(x, y, z) { return (y ^ (x | (~z))) }
  function FF(a, b, c, d, x, s, ac) { a = addUnsigned(a, addUnsigned(addUnsigned(F(b, c, d), x), ac)); return addUnsigned(rotateLeft(a, s), b) }
  function GG(a, b, c, d, x, s, ac) { a = addUnsigned(a, addUnsigned(addUnsigned(G(b, c, d), x), ac)); return addUnsigned(rotateLeft(a, s), b) }
  function HH(a, b, c, d, x, s, ac) { a = addUnsigned(a, addUnsigned(addUnsigned(H(b, c, d), x), ac)); return addUnsigned(rotateLeft(a, s), b) }
  function II(a, b, c, d, x, s, ac) { a = addUnsigned(a, addUnsigned(addUnsigned(I(b, c, d), x), ac)); return addUnsigned(rotateLeft(a, s), b) }
  function convertToWordArray(string) {
    let lWordCount
    const lMessageLength = string.length
    const lNumberOfWords_temp1 = lMessageLength + 8
    const lNumberOfWords_temp2 = (lNumberOfWords_temp1 - (lNumberOfWords_temp1 % 64)) / 64
    const lNumberOfWords = (lNumberOfWords_temp2 + 1) * 16
    const lWordArray = Array(lNumberOfWords - 1)
    let lBytePosition = 0
    let lByteCount = 0
    while (lByteCount < lMessageLength) {
      lWordCount = (lByteCount - (lByteCount % 4)) / 4
      lBytePosition = (lByteCount % 4) * 8
      lWordArray[lWordCount] = (lWordArray[lWordCount] | (string.charCodeAt(lByteCount) << lBytePosition))
      lByteCount++
    }
    lWordCount = (lByteCount - (lByteCount % 4)) / 4
    lBytePosition = (lByteCount % 4) * 8
    lWordArray[lWordCount] = lWordArray[lWordCount] | (0x80 << lBytePosition)
    lWordArray[lNumberOfWords - 2] = lMessageLength << 3
    lWordArray[lNumberOfWords - 1] = lMessageLength >>> 29
    return lWordArray
  }
  function wordToHex(lValue) {
    let WordToHexValue = '', WordToHexValue_temp = '', lByte, lCount
    for (lCount = 0; lCount <= 3; lCount++) {
      lByte = (lValue >>> (lCount * 8)) & 255
      WordToHexValue_temp = '0' + lByte.toString(16)
      WordToHexValue = WordToHexValue + WordToHexValue_temp.substr(WordToHexValue_temp.length - 2, 2)
    }
    return WordToHexValue
  }

  let x = []
  let k, AA, BB, CC, DD, a, b, c, d
  const S11 = 7, S12 = 12, S13 = 17, S14 = 22
  const S21 = 5, S22 = 9, S23 = 14, S24 = 20
  const S31 = 4, S32 = 11, S33 = 16, S34 = 23
  const S41 = 6, S42 = 10, S43 = 15, S44 = 21
  x = convertToWordArray(string)
  a = 0x67452301; b = 0xEFCDAB89; c = 0x98BADCFE; d = 0x10325476
  for (k = 0; k < x.length; k += 16) {
    AA = a; BB = b; CC = c; DD = d
    a = FF(a, b, c, d, x[k + 0], S11, 0xD76AA478)
    d = FF(d, a, b, c, x[k + 1], S12, 0xE8C7B756)
    c = FF(c, d, a, b, x[k + 2], S13, 0x242070DB)
    b = FF(b, c, d, a, x[k + 3], S14, 0xC1BDCEEE)
    a = FF(a, b, c, d, x[k + 4], S11, 0xF57C0FAF)
    d = FF(d, a, b, c, x[k + 5], S12, 0x4787C62A)
    c = FF(c, d, a, b, x[k + 6], S13, 0xA8304613)
    b = FF(b, c, d, a, x[k + 7], S14, 0xFD469501)
    a = FF(a, b, c, d, x[k + 8], S11, 0x698098D8)
    d = FF(d, a, b, c, x[k + 9], S12, 0x8B44F7AF)
    c = FF(c, d, a, b, x[k + 10], S13, 0xFFFF5BB1)
    b = FF(b, c, d, a, x[k + 11], S14, 0x895CD7BE)
    a = FF(a, b, c, d, x[k + 12], S11, 0x6B901122)
    d = FF(d, a, b, c, x[k + 13], S12, 0xFD987193)
    c = FF(c, d, a, b, x[k + 14], S13, 0xA679438E)
    b = FF(b, c, d, a, x[k + 15], S14, 0x49B40821)
    a = GG(a, b, c, d, x[k + 1], S21, 0xF61E2562)
    d = GG(d, a, b, c, x[k + 6], S22, 0xC040B340)
    c = GG(c, d, a, b, x[k + 11], S23, 0x265E5A51)
    b = GG(b, c, d, a, x[k + 0], S24, 0xE9B6C7AA)
    a = GG(a, b, c, d, x[k + 5], S21, 0xD62F105D)
    d = GG(d, a, b, c, x[k + 10], S22, 0x2441453)
    c = GG(c, d, a, b, x[k + 15], S23, 0xD8A1E681)
    b = GG(b, c, d, a, x[k + 4], S24, 0xE7D3FBC8)
    a = GG(a, b, c, d, x[k + 9], S21, 0x21E1CDE6)
    d = GG(d, a, b, c, x[k + 14], S22, 0xC33707D6)
    c = GG(c, d, a, b, x[k + 3], S23, 0xF4D50D87)
    b = GG(b, c, d, a, x[k + 8], S24, 0x455A14ED)
    a = GG(a, b, c, d, x[k + 13], S21, 0xA9E3E905)
    d = GG(d, a, b, c, x[k + 2], S22, 0xFCEFA3F8)
    c = GG(c, d, a, b, x[k + 7], S23, 0x676F02D9)
    b = GG(b, c, d, a, x[k + 12], S24, 0x8D2A4C8A)
    a = HH(a, b, c, d, x[k + 5], S31, 0xFFFA3942)
    d = HH(d, a, b, c, x[k + 8], S32, 0x8771F681)
    c = HH(c, d, a, b, x[k + 11], S33, 0x6D9D6122)
    b = HH(b, c, d, a, x[k + 14], S34, 0xFDE5380C)
    a = HH(a, b, c, d, x[k + 1], S31, 0xA4BEEA44)
    d = HH(d, a, b, c, x[k + 4], S32, 0x4BDECFA9)
    c = HH(c, d, a, b, x[k + 7], S33, 0xF6BB4B60)
    b = HH(b, c, d, a, x[k + 10], S34, 0xBEBFBC70)
    a = HH(a, b, c, d, x[k + 13], S31, 0x289B7EC6)
    d = HH(d, a, b, c, x[k + 0], S32, 0xEAA127FA)
    c = HH(c, d, a, b, x[k + 3], S33, 0xD4EF3085)
    b = HH(b, c, d, a, x[k + 6], S34, 0x4881D05)
    a = HH(a, b, c, d, x[k + 9], S31, 0xD9D4D039)
    d = HH(d, a, b, c, x[k + 12], S32, 0xE6DB99E5)
    c = HH(c, d, a, b, x[k + 15], S33, 0x1FA27CF8)
    b = HH(b, c, d, a, x[k + 2], S34, 0xC4AC5665)
    a = II(a, b, c, d, x[k + 0], S41, 0xF4292244)
    d = II(d, a, b, c, x[k + 7], S42, 0x432AFF97)
    c = II(c, d, a, b, x[k + 14], S43, 0xAB9423A7)
    b = II(b, c, d, a, x[k + 5], S44, 0xFC93A039)
    a = II(a, b, c, d, x[k + 12], S41, 0x655B59C3)
    d = II(d, a, b, c, x[k + 3], S42, 0x8F0CCC92)
    c = II(c, d, a, b, x[k + 10], S43, 0xFFEFF47D)
    b = II(b, c, d, a, x[k + 1], S44, 0x85845DD1)
    a = II(a, b, c, d, x[k + 8], S41, 0x6FA87E4F)
    d = II(d, a, b, c, x[k + 15], S42, 0xFE2CE6E0)
    c = II(c, d, a, b, x[k + 6], S43, 0xA3014314)
    b = II(b, c, d, a, x[k + 13], S44, 0x4E0811A1)
    a = II(a, b, c, d, x[k + 4], S41, 0xF7537E82)
    d = II(d, a, b, c, x[k + 11], S42, 0xBD3AF235)
    c = II(c, d, a, b, x[k + 2], S43, 0x2AD7D2BB)
    b = II(b, c, d, a, x[k + 9], S44, 0xEB86D391)
    a = addUnsigned(a, AA)
    b = addUnsigned(b, BB)
    c = addUnsigned(c, CC)
    d = addUnsigned(d, DD)
  }
  return (wordToHex(a) + wordToHex(b) + wordToHex(c) + wordToHex(d)).toLowerCase()
}
</script>

<style scoped>
.hash-file { display: flex; flex-direction: column; gap: 16px; }
.hash-file__dropzone { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; padding: 40px; border: 2px dashed var(--border-color); border-radius: 12px; background: var(--bg-surface); cursor: pointer; transition: all 0.2s; }
.hash-file__dropzone:hover { border-color: var(--color-primary); background: color-mix(in srgb, var(--color-primary) 5%, transparent); }
.hash-file__dropzone p { font-size: 14px; color: var(--text-secondary); }
.hash-file__dropzone small { font-size: 12px; color: var(--text-tertiary); }
.hash-file__results { display: flex; flex-direction: column; gap: 8px; }
.hash-file__result { display: flex; align-items: center; gap: 10px; padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.hash-file__algo { font-size: 12px; font-weight: 600; color: var(--color-primary); min-width: 70px; }
.hash-file__hash { flex: 1; font-family: monospace; font-size: 12px; color: var(--text-secondary); word-break: break-all; }
.hash-file__copy { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-secondary); font-size: 11px; cursor: pointer; }
.hash-file__copy:hover { border-color: var(--color-primary); color: var(--color-primary); }
.hash-file__progress { text-align: center; padding: 12px; color: var(--color-primary); font-size: 14px; }
</style>
