<template>
  <h-single-layout>
    <div class="pem">
      <div class="pem__field"><label>输入 (Base64 或 PEM)</label><textarea v-model="input" rows="5" spellcheck="false"></textarea></div>
      <div class="pem__actions">
        <select v-model="pemType">
          <option value="RSA PRIVATE KEY">RSA PRIVATE KEY</option>
          <option value="RSA PUBLIC KEY">RSA PUBLIC KEY</option>
          <option value="EC PRIVATE KEY">EC PRIVATE KEY</option>
          <option value="PUBLIC KEY">PUBLIC KEY</option>
          <option value="CERTIFICATE">CERTIFICATE</option>
          <option value="PRIVATE KEY">PRIVATE KEY</option>
        </select>
        <label class="pem__chk"><input type="checkbox" v-model="toSingle" /> 去换行(单行)</label>
        <label class="pem__chk"><input type="checkbox" v-model="toBase64" /> 提取纯 Base64</label>
      </div>
      <div class="pem__output selectable">{{ output }}</div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref, computed } from 'vue'
const input = ref('MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0123456789abcdef')
const pemType = ref('RSA PRIVATE KEY')
const toSingle = ref(false)
const toBase64 = ref(false)

const output = computed(() => {
  let raw = input.value.replace(/-----BEGIN [^-]+-----/g,'').replace(/-----END [^-]+-----/g,'').replace(/\s/g,'')
  if (toBase64.value) return raw
  if (toSingle.value) return `-----BEGIN ${pemType.value}-----\n${raw}\n-----END ${pemType.value}-----`
  const lines = raw.match(/.{1,64}/g) || []
  return `-----BEGIN ${pemType.value}-----\n${lines.join('\n')}\n-----END ${pemType.value}-----`
})
</script>
<style scoped>
.pem{display:flex;flex-direction:column;gap:12px}
.pem__field{display:flex;flex-direction:column;gap:4px}
.pem__field label{font-size:12px;color:var(--text-tertiary)}
.pem__field textarea{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;font-size:12px;outline:none;resize:vertical}
.pem__actions{display:flex;align-items:center;gap:12px;flex-wrap:wrap}
.pem__actions select{padding:6px 10px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-primary)}
.pem__chk{font-size:13px;color:var(--text-secondary);display:flex;align-items:center;gap:4px;cursor:pointer}
.pem__output{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base);font-family:monospace;font-size:12px;min-height:100px;white-space:pre-wrap;word-break:break-all}
</style>
