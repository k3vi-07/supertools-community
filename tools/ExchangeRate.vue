<template>
  <h-single-layout>
    <div class="exchange">
      <div class="exchange__amount"><label>金额</label><input type="number" v-model.number="amount" step="0.01" /><select v-model="base"><option v-for="c in currencies" :key="c" :value="c">{{ c }}</option></select></div>
      <button class="exchange__btn" @click="fetchRates">查询汇率</button>
      <div v-if="loading" class="exchange__loading">查询中...</div>
      <div v-if="rates" class="exchange__rates">
        <div v-for="c in currencies.filter(x=>x!==base)" :key="c" class="exchange__rate">
          <span class="exchange__ccy">{{ c }}</span>
          <span class="exchange__val selectable">{{ (amount * (rates[c] || 0)).toFixed(2) }}</span>
          <span class="exchange__rate-val">1 {{ base }} = {{ (rates[c]||0).toFixed(4) }} {{ c }}</span>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const currencies = ['USD','EUR','CNY','JPY','GBP','KRW','HKD','AUD','CAD','SGD']
const amount = ref(100)
const base = ref('USD')
const rates = ref(null)
const loading = ref(false)

async function fetchRates() {
  loading.value = true
  try {
    const src = await window.supertools.fetchRemote(`https://api.exchangerate-api.com/v4/latest/${base.value}`)
    const data = JSON.parse(src)
    rates.value = {}
    for (const c of currencies) {
      if (c !== base.value && data.rates[c]) rates.value[c] = data.rates[c]
    }
  } catch (err) {
    window.$he3?.message.error('查询失败: ' + err.message)
  } finally { loading.value = false }
}
fetchRates()
</script>

<style scoped>
.exchange { display: flex; flex-direction: column; gap: 16px; }
.exchange__amount { display: flex; align-items: flex-end; gap: 8px; }
.exchange__amount label { font-size: 12px; color: var(--text-secondary); display: block; margin-bottom: 4px; }
.exchange__amount input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 16px; width: 120px; }
.exchange__amount select { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.exchange__btn { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
.exchange__loading { text-align: center; color: var(--text-tertiary); }
.exchange__rates { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 8px; }
.exchange__rate { display: flex; flex-direction: column; gap: 4px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.exchange__ccy { font-size: 14px; font-weight: 700; color: var(--text-primary); }
.exchange__val { font-size: 20px; font-weight: 700; color: var(--color-primary); }
.exchange__rate-val { font-size: 11px; color: var(--text-tertiary); }
</style>
