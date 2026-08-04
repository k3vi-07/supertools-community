<template>
  <h-single-layout>
    <div class="loan-calc">
      <div class="loan-calc__grid">
        <div class="loan-calc__field"><label>贷款金额</label><input type="number" v-model.number="principal" step="1000" /></div>
        <div class="loan-calc__field"><label>年利率 (%)</label><input type="number" v-model.number="annualRate" step="0.01" /></div>
        <div class="loan-calc__field"><label>贷款年限</label><input type="number" v-model.number="years" step="1" /></div>
      </div>
      <div class="loan-calc__results">
        <div class="loan-calc__card"><span class="loan-calc__label">月供</span><span class="loan-calc__value">{{ formatCurrency(monthlyPayment) }}</span></div>
        <div class="loan-calc__card"><span class="loan-calc__label">总利息</span><span class="loan-calc__value">{{ formatCurrency(totalInterest) }}</span></div>
        <div class="loan-calc__card"><span class="loan-calc__label">总还款</span><span class="loan-calc__value">{{ formatCurrency(totalPayment) }}</span></div>
        <div class="loan-calc__card"><span class="loan-calc__label">利息占比</span><span class="loan-calc__value">{{ interestRatio }}%</span></div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const principal = ref(1000000)
const annualRate = ref(4.9)
const years = ref(30)

const monthlyPayment = computed(() => {
  const r = annualRate.value / 100 / 12
  const n = years.value * 12
  if (r === 0) return principal.value / n
  return principal.value * r * Math.pow(1 + r, n) / (Math.pow(1 + r, n) - 1)
})
const totalPayment = computed(() => monthlyPayment.value * years.value * 12)
const totalInterest = computed(() => totalPayment.value - principal.value)
const interestRatio = computed(() => ((totalInterest.value / totalPayment.value) * 100).toFixed(1))

function formatCurrency(n: number): string {
  return '¥' + n.toLocaleString('zh-CN', { maximumFractionDigits: 2 })
}
</script>

<style scoped>
.loan-calc { display: flex; flex-direction: column; gap: 16px; }
.loan-calc__grid { display: flex; gap: 12px; }
.loan-calc__field { flex: 1; display: flex; flex-direction: column; gap: 4px; }
.loan-calc__field label { font-size: 12px; color: var(--text-secondary); }
.loan-calc__field input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.loan-calc__results { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
.loan-calc__card { display: flex; flex-direction: column; gap: 4px; padding: 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.loan-calc__label { font-size: 12px; color: var(--text-tertiary); }
.loan-calc__value { font-size: 20px; font-weight: 700; color: var(--color-primary); }
</style>
