<template>
  <h-single-layout>
    <div class="ecdh">
      <div class="ecdh__intro">ECDH 密钥交换 — 双方各自生成密钥对，交换公钥后可推导出相同的共享密钥。</div>
      <div class="ecdh__cols">
        <div class="ecdh__party">
          <div class="ecdh__party-title">Alice</div>
          <div class="ecdh__kv"><span>私钥</span><pre class="ecdh__key selectable">{{ alice.priv || '—' }}</pre></div>
          <div class="ecdh__kv"><span>公钥</span><pre class="ecdh__key selectable">{{ alice.pub || '—' }}</pre></div>
        </div>
        <div class="ecdh__party">
          <div class="ecdh__party-title">Bob</div>
          <div class="ecdh__kv"><span>私钥</span><pre class="ecdh__key selectable">{{ bob.priv || '—' }}</pre></div>
          <div class="ecdh__kv"><span>公钥</span><pre class="ecdh__key selectable">{{ bob.pub || '—' }}</pre></div>
        </div>
      </div>
      <div class="ecdh__actions">
        <button class="ecdh__btn" @click="exchange" :disabled="loading">{{ loading ? '交换中...' : '🔄 执行密钥交换' }}</button>
      </div>
      <div v-if="aliceShared" class="ecdh__result">
        <div class="ecdh__result-label">共享密钥对比</div>
        <div class="ecdh__kv"><span>Alice 推导</span><code class="ecdh__shared selectable">{{ aliceShared }}</code></div>
        <div class="ecdh__kv"><span>Bob 推导</span><code class="ecdh__shared selectable">{{ bobShared }}</code></div>
        <div :class="['ecdh__match', aliceShared === bobShared ? 'ok' : 'err']">
          {{ aliceShared === bobShared ? '✅ 双方共享密钥一致！' : '❌ 密钥不匹配' }}
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const loading = ref(false)
const alice = ref({ priv: '', pub: '' })
const bob = ref({ priv: '', pub: '' })
const aliceShared = ref('')
const bobShared = ref('')

const buf2hex = (b) => [...new Uint8Array(b)].map(x => x.toString(16).padStart(2,'0')).join('')

async function exchange() {
  loading.value = true
  try {
    // Alice generates key pair
    const aliceKp = await crypto.subtle.generateKey({ name: 'ECDH', namedCurve: 'P-256' }, true, ['deriveKey','deriveBits'])
    alice.value.priv = buf2hex(await crypto.subtle.exportKey('pkcs8', aliceKp.privateKey).then(buf2hex).catch(()=>''))
    alice.value.priv = (await crypto.subtle.exportKey('pkcs8', aliceKp.privateKey)).toString()
    const aPriv = await crypto.subtle.exportKey('pkcs8', aliceKp.privateKey)
    const aPub = await crypto.subtle.exportKey('spki', aliceKp.publicKey)
    alice.value.priv = buf2hex(aPriv)
    alice.value.pub = buf2hex(aPub)

    // Bob generates key pair
    const bobKp = await crypto.subtle.generateKey({ name: 'ECDH', namedCurve: 'P-256' }, true, ['deriveKey','deriveBits'])
    const bPriv = await crypto.subtle.exportKey('pkcs8', bobKp.privateKey)
    const bPub = await crypto.subtle.exportKey('spki', bobKp.publicKey)
    bob.value.priv = buf2hex(bPriv)
    bob.value.pub = buf2hex(bPub)

    // Alice derives shared key using Bob's public key
    const aSharedBuf = await crypto.subtle.deriveBits(
      { name: 'ECDH', public: bobKp.publicKey },
      aliceKp.privateKey,
      256
    )
    aliceShared.value = buf2hex(aSharedBuf)

    // Bob derives shared key using Alice's public key
    const bSharedBuf = await crypto.subtle.deriveBits(
      { name: 'ECDH', public: aliceKp.publicKey },
      bobKp.privateKey,
      256
    )
    bobShared.value = buf2hex(bSharedBuf)

    window.$he3?.message.success('密钥交换完成')
  } catch(e) { window.$he3?.message.error(e.message) }
  loading.value = false
}
</script>
<style scoped>
.ecdh { display:flex; flex-direction:column; gap:14px; }
.ecdh__intro { font-size:13px; color:var(--text-tertiary); padding:10px; border-radius:8px; background:var(--bg-surface); }
.ecdh__cols { display:grid; grid-template-columns:1fr 1fr; gap:12px; }
.ecdh__party { padding:12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); }
.ecdh__party-title { font-size:14px; font-weight:700; color:var(--color-primary); margin-bottom:8px; }
.ecdh__kv { display:flex; flex-direction:column; gap:2px; margin-bottom:8px; }
.ecdh__kv span { font-size:11px; color:var(--text-tertiary); }
.ecdh__key { font-size:10px; color:var(--text-secondary); word-break:break-all; max-height:60px; overflow:auto; margin:0; }
.ecdh__btn { padding:12px; border:none; border-radius:8px; background:var(--color-primary); color:white; cursor:pointer; font-size:14px; font-weight:600; }
.ecdh__btn:disabled{opacity:0.5;}
.ecdh__result { padding:12px; border:1px solid var(--border-color); border-radius:8px; background:var(--bg-surface); }
.ecdh__result-label { font-size:13px; font-weight:600; color:var(--text-primary); margin-bottom:8px; }
.ecdh__shared { font-size:11px; word-break:break-all; color:var(--text-secondary); }
.ecdh__match { margin-top:8px; padding:8px; border-radius:6px; font-weight:600; text-align:center; }
.ecdh__match.ok { background:rgba(34,197,94,0.15); color:#22c55e; }
.ecdh__match.err { background:rgba(239,68,68,0.15); color:#ef4444; }
</style>
