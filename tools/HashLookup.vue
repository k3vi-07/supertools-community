<template>
  <h-single-layout>
    <div class="hash-lookup">
      <div class="hash-lookup__input"><label>输入哈希值</label><input v-model="hash" placeholder="粘贴哈希值..." /></div>
      <div v-if="info" class="hash-lookup__result">
        <div class="hash-lookup__row"><span>类型</span><strong>{{ info.type }}</strong></div>
        <div class="hash-lookup__row"><span>长度</span><code>{{ info.length }} 字符 ({{ info.bits }} 位)</code></div>
        <div class="hash-lookup__row"><span>十六进制</span><code>{{ info.isHex ? '✅ 是' : '❌ 否' }}</code></div>
        <div v-if="info.type !== 'Unknown'" class="hash-lookup__row"><span>可能算法</span><strong>{{ info.algo }}</strong></div>
        <div class="hash-lookup__row"><span>彩虹表风险</span><strong :class="info.riskClass">{{ info.risk }}</strong></div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const hash = ref('d41d8cd98f00b204e9800998ecf8427e')

const info = computed(() => {
  const h = hash.value.trim()
  if (!h) return null
  const isHex = /^[0-9a-fA-F]+$/.test(h)
  const length = h.length
  const types: Record<number, { type: string; algo: string; bits: number }> = {
    32: { type: 'MD5', algo: 'MD5 / NTLM', bits: 128 },
    40: { type: 'SHA-1', algo: 'SHA-1 / SHA-0 / RIPEMD-160 / MySQL5(20+)', bits: 160 },
    56: { type: 'SHA-224', algo: 'SHA-224', bits: 224 },
    64: { type: 'SHA-256', algo: 'SHA-256 / SHA3-256 / BLAKE2s / SM3', bits: 256 },
    96: { type: 'SHA-384', algo: 'SHA-384', bits: 384 },
    128: { type: 'SHA-512', algo: 'SHA-512 / BLAKE2b', bits: 512 }
  }
  const t = types[length]
  const risk = length <= 32 ? '🔴 高（易破解）' : length <= 40 ? '🟡 中' : '🟢 低'
  const riskClass = length <= 32 ? 'risk-high' : length <= 40 ? 'risk-mid' : 'risk-low'
  return {
    type: t?.type || 'Unknown',
    algo: t?.algo || '未知',
    bits: t?.bits || length * 4,
    length, isHex,
    risk, riskClass
  }
})
</script>

<style scoped>
.hash-lookup { display: flex; flex-direction: column; gap: 16px; }
.hash-lookup__input { display: flex; flex-direction: column; gap: 4px; }
.hash-lookup__input label { font-size: 12px; color: var(--text-secondary); }
.hash-lookup__input input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; outline: none; }
.hash-lookup__result { display: flex; flex-direction: column; gap: 8px; padding: 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.hash-lookup__row { display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px solid var(--border-color-light); font-size: 13px; }
.hash-lookup__row span { color: var(--text-tertiary); }
.hash-lookup__row code { font-family: monospace; color: var(--color-primary); }
.risk-high { color: var(--color-error); }
.risk-mid { color: var(--color-warning); }
.risk-low { color: var(--color-success); }
</style>
