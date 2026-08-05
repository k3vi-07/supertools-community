<template>
  <h-single-layout>
    <div class="compound">
      <div class="compound__grid">
        <div class="compound__field"><label>本金</label><input type="number" v-model.number="principal" /></div>
        <div class="compound__field"><label>年利率 (%)</label><input type="number" v-model.number="rate" step="0.1" /></div>
        <div class="compound__field"><label>年限</label><input type="number" v-model.number="years" /></div>
      </div>
      <div class="compound__results">
        <div class="compound__card"><span>到期金额</span><strong>{{ fmt(finalAmount) }}</strong></div>
        <div class="compound__card"><span>总收益</span><strong>{{ fmt(profit) }}</strong></div>
        <div class="compound__card"><span>收益率</span><strong>{{ roi }}%</strong></div>
        <div class="compound__card"><span>翻倍时间</span><strong>{{ doubleTime }} 年</strong></div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const principal = ref(10000)
const rate = ref(5)
const years = ref(10)

const finalAmount = computed(() => principal.value * Math.pow(1 + rate.value / 100, years.value))
const profit = computed(() => finalAmount.value - principal.value)
const roi = computed(() => ((profit.value / principal.value) * 100).toFixed(1))
const doubleTime = computed(() => rate.value > 0 ? (72 / rate.value).toFixed(1) : '∞')

function fmt(n) {
  return '¥' + n.toLocaleString('zh-CN', { maximumFractionDigits: 2 })
}
</script>

<style scoped>
.compound { display: flex; flex-direction: column; gap: 16px; }
.compound__grid { display: flex; gap: 12px; }
.compound__field { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.compound__field label { font-size: 12px; color: var(--text-secondary); }
.compound__field input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.compound__results { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
.compound__card { display: flex; flex-direction: column; gap: 4px; padding: 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.compound__card span { font-size: 12px; color: var(--text-tertiary); }
.compound__card strong { font-size: 20px; color: var(--color-primary); }
</style>
