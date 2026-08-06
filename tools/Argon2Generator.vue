<template>
  <h-single-layout>
    <div class="argon2">
      <div class="argon2__intro">⚠️ 浏览器环境无法实现完整 Argon2（需要大量内存计算），此工具展示参数配置和 PBKDF2 替代方案。</div>
      <div class="argon2__field"><label>密码</label><input v-model="password" type="text" /></div>
      <div class="argon2__field"><label>Salt (Hex)</label><input v-model="salt" /></div>
      <div class="argon2__params">
        <div class="argon2__param"><label>迭代次数</label><input type="number" v-model.number="iterations" min="1" /></div>
        <div class="argon2__param"><label>输出长度 (字节)</label><input type="number" v-model.number="keyLen" min="16" /></div>
        <div class="argon2__param"><label>哈希算法</label><select v-model="hash"><option value="SHA-256">SHA-256</option><option value="SHA-384">SHA-384</option><option value="SHA-512">SHA-512</option></select></div>
      </div>
      <button class="argon2__btn" @click="derive" :disabled="loading">{{ loading?'计算中...':'派生密钥' }}</button>
      <div v-if="result" class="argon2__result">
        <div class="argon2__label">派生密钥 (Hex)</div>
        <div class="argon2__value selectable">{{ result }}</div>
        <div class="argon2__label">Base64</div>
        <div class="argon2__value selectable">{{ resultB64 }}</div>
      </div>
      <div class="argon2__config">
        <div class="argon2__label">Argon2 推荐参数：</div>
        <div class="argon2__kv"><span>t (迭代)</span><code>3</code></div>
        <div class="argon2__kv"><span>m (内存 MB)</span><code>65536 (64MB)</code></div>
        <div class="argon2__kv"><span>p (并行)</span><code>4</code></div>
        <div class="argon2__kv"><span>type</span><code>argon2id</code></div>
      </div>
    </div>
  </h-single-layout>
</template>
<script setup>
import { ref } from 'vue'
const password = ref('mypassword')
const salt = ref('0123456789abcdef')
const iterations = ref(100000)
const keyLen = ref(32)
const hash = ref('SHA-256')
const result = ref('')
const resultB64 = ref('')
const loading = ref(false)

async function derive() {
  loading.value = true
  try {
    const enc = new TextEncoder()
    const keyMaterial = await crypto.subtle.importKey('raw', enc.encode(password.value), 'PBKDF2', false, ['deriveBits'])
    const saltBytes = new Uint8Array(salt.value.length/2)
    for (let i=0;i<saltBytes.length;i++) saltBytes[i]=parseInt(salt.value.substr(i*2,2),16)
    const bits = await crypto.subtle.deriveBits(
      { name:'PBKDF2', salt:saltBytes, iterations:iterations.value, hash:hash.value },
      keyMaterial, keyLen.value*8
    )
    const bytes = new Uint8Array(bits)
    result.value = [...bytes].map(b=>b.toString(16).padStart(2,'0')).join('')
    resultB64.value = btoa(String.fromCharCode(...bytes))
    window.$he3?.message.success('密钥派生完成')
  } catch(e) { window.$he3?.message.error(e.message) }
  loading.value = false
}
</script>
<style scoped>
.argon2{display:flex;flex-direction:column;gap:12px}
.argon2__intro{padding:10px;border-radius:8px;background:rgba(245,158,11,0.1);color:#f59e0b;font-size:12px}
.argon2__field{display:flex;flex-direction:column;gap:4px}
.argon2__field label{font-size:12px;color:var(--text-tertiary)}
.argon2__field input{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;outline:none}
.argon2__params{display:grid;grid-template-columns:repeat(3,1fr);gap:8px}
.argon2__param{display:flex;flex-direction:column;gap:4px}
.argon2__param label{font-size:12px;color:var(--text-tertiary)}
.argon2__param input,.argon2__param select{padding:6px 8px;border:1px solid var(--border-color);border-radius:6px;background:var(--bg-surface);color:var(--text-primary);outline:none}
.argon2__btn{padding:12px;border:none;border-radius:8px;background:var(--color-primary);color:white;cursor:pointer;font-weight:600}
.argon2__btn:disabled{opacity:0.5}
.argon2__result{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface)}
.argon2__label{font-size:12px;color:var(--text-tertiary);margin-bottom:4px;margin-top:8px}
.argon2__label:first-child{margin-top:0}
.argon2__value{font-family:monospace;font-size:13px;color:var(--color-primary);word-break:break-all}
.argon2__config{padding:12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-base)}
.argon2__kv{display:flex;justify-content:space-between;font-size:13px;padding:4px 0}
.argon2__kv span{color:var(--text-tertiary)}
.argon2__kv code{color:var(--text-primary)}
</style>
