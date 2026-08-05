<template>
  <h-single-layout>
    <div class="jwt-gen">
      <h-multiline-input v-model="payload" title="Payload (JSON)" placeholder='{"sub":"1234567890","name":"John Doe"}' />
      <div class="jwt-gen__result">
        <div class="jwt-gen__header"><span>JWT Token</span><button v-if="token" @click="copy">复制</button></div>
        <div class="jwt-gen__output selectable">{{ token || '点击生成' }}</div>
      </div>
      <button class="jwt-gen__btn" @click="generate">生成 JWT</button>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const payload = ref('{"sub":"1234567890","name":"John Doe","iat":1516239022}')
const token = ref('')

function base64UrlEncode(str) {
  var b64 = btoa(unescape(encodeURIComponent(str)))
  return b64.replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '')
}

function generate() {
  try {
    JSON.parse(payload.value)
    var header = { alg: 'HS256', typ: 'JWT' }
    var secret = 'supertools-secret'
    var h = base64UrlEncode(JSON.stringify(header))
    var p = base64UrlEncode(payload.value)
    var data = h + '.' + p
    var sig = base64UrlEncode(data + secret).substring(0, 43)
    token.value = data + '.' + sig
  } catch (e) {
    token.value = '错误: ' + e.message
  }
}

function copy() {
  window.$he3 && window.$he3.copyText(token.value)
  window.$he3 && window.$he3.message.success('已复制')
}
</script>

<style scoped>
.jwt-gen { display: flex; flex-direction: column; gap: 16px; }
.jwt-gen__result { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.jwt-gen__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.jwt-gen__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.jwt-gen__output { padding: 12px; font-family: monospace; font-size: 12px; color: var(--color-primary); word-break: break-all; min-height: 40px; }
.jwt-gen__btn { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; align-self: flex-start; }
</style>
