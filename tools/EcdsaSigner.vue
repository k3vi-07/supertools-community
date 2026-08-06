<template>
  <h-single-layout>
    <div class="ecdsa">
      <div class="ecdsa__tabs">
        <button :class="{active:tab==='keypair'}" @click="tab='keypair'">生成密钥对</button>
        <button :class="{active:tab==='sign'}" @click="tab='sign'">签名</button>
        <button :class="{active:tab==='verify'}" @click="tab='verify'">验签</button>
      </div>
      <!-- 生成密钥对 -->
      <div v-if="tab==='keypair'" class="ecdsa__panel">
        <div class="ecdsa__row"><label>曲线</label><select v-model="curve"><option value="P-256">P-256 (secp256r1)</option><option value="P-384">P-384</option><option value="P-521">P-521</option></select></div>
        <button class="ecdsa__btn" @click="genKey" :disabled="loading">{{ loading ? '生成中...' : '生成密钥对' }}</button>
        <div v-if="publicKey" class="ecdsa__result">
          <div class="ecdsa__label">Public Key (SPKI Base64)</div>
          <pre class="ecdsa__code selectable">{{ publicKey }}</pre>
        </div>
        <div v-if="privateKey" class="ecdsa__result">
          <div class="ecdsa__label">Private Key (PKCS8 Base64)</div>
          <pre class="ecdsa__code selectable">{{ privateKey }}</pre>
        </div>
      </div>
      <!-- 签名 -->
      <div v-if="tab==='sign'" class="ecdsa__panel">
        <div class="ecdsa__row"><label>消息</label><textarea v-model="message" rows="3"></textarea></div>
        <div class="ecdsa__row"><label>私钥 (Base64)</label><textarea v-model="privKeyInput" rows="4" spellcheck="false"></textarea></div>
        <button class="ecdsa__btn" @click="sign" :disabled="loading">{{ loading ? '签名中...' : '签名' }}</button>
        <div v-if="signature" class="ecdsa__result"><div class="ecdsa__label">签名 (Base64)</div><pre class="ecdsa__code selectable">{{ signature }}</pre></div>
      </div>
      <!-- 验签 -->
      <div v-if="tab==='verify'" class="ecdsa__panel">
        <div class="ecdsa__row"><label>消息</label><textarea v-model="message" rows="3"></textarea></div>
        <div class="ecdsa__row"><label>公钥 (Base64)</label><textarea v-model="pubKeyInput" rows="4" spellcheck="false"></textarea></div>
        <div class="ecdsa__row"><label>签名 (Base64)</label><textarea v-model="sigInput" rows="3" spellcheck="false"></textarea></div>
        <button class="ecdsa__btn" @click="verify" :disabled="loading">{{ loading ? '验签中...' : '验证签名' }}</button>
        <div v-if="verifyResult !== null" class="ecdsa__result" :class="{ok:verifyResult,err:!verifyResult}">
          {{ verifyResult ? '✅ 签名验证通过' : '❌ 签名验证失败' }}
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const tab = ref('keypair')
const curve = ref('P-256')
const loading = ref(false)
const publicKey = ref('')
const privateKey = ref('')
const message = ref('Hello ECDSA!')
const privKeyInput = ref('')
const pubKeyInput = ref('')
const signature = ref('')
const sigInput = ref('')
const verifyResult = ref(null)

const b642buf = (s) => Uint8Array.from(atob(s), c => c.charCodeAt(0))
const buf2b64 = (b) => btoa(String.fromCharCode(...new Uint8Array(b)))

async function genKey() {
  loading.value = true
  try {
    const kp = await crypto.subtle.generateKey({ name: 'ECDSA', namedCurve: curve.value }, true, ['sign','verify'])
    const pub = await crypto.subtle.exportKey('spki', kp.publicKey)
    const priv = await crypto.subtle.exportKey('pkcs8', kp.privateKey)
    publicKey.value = buf2b64(pub)
    privateKey.value = buf2b64(priv)
    window.$he3?.message.success('密钥对已生成')
  } catch(e) { window.$he3?.message.error(e.message) }
  loading.value = false
}

async function sign() {
  loading.value = true
  try {
    const key = await crypto.subtle.importKey('pkcs8', b642buf(privKeyInput.value), { name:'ECDSA', namedCurve: curve.value }, false, ['sign'])
    const sig = await crypto.subtle.sign({ name:'ECDSA', hash:'SHA-256' }, key, new TextEncoder().encode(message.value))
    signature.value = buf2b64(sig)
    window.$he3?.message.success('签名成功')
  } catch(e) { window.$he3?.message.error(e.message) }
  loading.value = false
}

async function verify() {
  loading.value = true; verifyResult.value = null
  try {
    const key = await crypto.subtle.importKey('spki', b642buf(pubKeyInput.value), { name:'ECDSA', namedCurve: curve.value }, false, ['verify'])
    const valid = await crypto.subtle.verify({ name:'ECDSA', hash:'SHA-256' }, key, b642buf(sigInput.value), new TextEncoder().encode(message.value))
    verifyResult.value = valid
  } catch(e) { verifyResult.value = false }
  loading.value = false
}
</script>
<style scoped>
.ecdsa { display:flex; flex-direction:column; gap:12px; }
.ecdsa__tabs { display:flex; }
.ecdsa__tabs button { flex:1; padding:8px; border:1px solid var(--border-color); background:var(--bg-surface); color:var(--text-secondary); cursor:pointer; }
.ecdsa__tabs button:first-child{border-radius:8px 0 0 8px;} .ecdsa__tabs button:last-child{border-radius:0 8px 8px 0;border-left:none;}
.ecdsa__tabs button:not(:first-child):not(:last-child){border-left:none;}
.ecdsa__tabs button.active{background:var(--color-primary);color:white;border-color:var(--color-primary);}
.ecdsa__panel{display:flex;flex-direction:column;gap:10px;}
.ecdsa__row{display:flex;flex-direction:column;gap:4px;}
.ecdsa__row label{font-size:12px;color:var(--text-tertiary);}
.ecdsa__row select,.ecdsa__row textarea,.ecdsa__row input{padding:8px 12px;border:1px solid var(--border-color);border-radius:8px;background:var(--bg-surface);color:var(--text-primary);font-family:monospace;font-size:13px;outline:none;resize:vertical;}
.ecdsa__btn{padding:10px;border:none;border-radius:8px;background:var(--color-primary);color:white;cursor:pointer;font-weight:600;}
.ecdsa__btn:disabled{opacity:0.5;}
.ecdsa__result{padding:8px;}
.ecdsa__label{font-size:12px;color:var(--text-tertiary);margin-bottom:4px;}
.ecdsa__code{padding:12px;border-radius:8px;background:var(--bg-base);font-size:12px;word-break:break-all;white-space:pre-wrap;}
.ecdsa__result.ok{color:#22c55e;font-weight:600;}
.ecdsa__result.err{color:#ef4444;font-weight:600;}
</style>
