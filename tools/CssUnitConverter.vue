<template>
  <h-single-layout>
    <div class="css-unit">
      <div class="css-unit__row">
        <div class="css-unit__field">
          <label>基准像素 (px)</label>
          <input type="number" v-model.number="basePx" min="1" />
        </div>
        <div class="css-unit__field">
          <label>根字号 (root px)</label>
          <input type="number" v-model.number="rootSize" min="1" />
        </div>
      </div>
      <div class="css-unit__converter">
        <div class="css-unit__input-group">
          <input v-model="inputVal" type="number" placeholder="输入数值" @input="convert" />
          <select v-model="fromUnit" @change="convert">
            <option value="px">px</option>
            <option value="rem">rem</option>
            <option value="em">em</option>
            <option value="pt">pt</option>
            <option value="vw">vw</option>
            <option value="vh">vh</option>
            <option value="%">%</option>
          </select>
        </div>
        <span class="css-unit__arrow">=</span>
        <div class="css-unit__results">
          <div v-for="u in units" :key="u.id" class="css-unit__result" @click="copy(u.value + u.unit)">
            <span class="css-unit__num">{{ u.value }}</span>
            <span class="css-unit__unit">{{ u.unit }}</span>
          </div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const basePx = ref(16)
const rootSize = ref(16)
const inputVal = ref(16)
const fromUnit = ref('px')

const units = computed(() => {
  const val = parseFloat(inputVal.value) || 0
  if (val === 0) return []
  // 先转成 px
  let px
  switch (fromUnit.value) {
    case 'px': px = val; break
    case 'rem': px = val * rootSize.value; break
    case 'em': px = val * rootSize.value; break
    case 'pt': px = val * 1.333333; break
    case 'vw': px = val * (window.innerWidth / 100); break
    case 'vh': px = val * (window.innerHeight / 100); break
    case '%': px = val * basePx.value / 100; break
    default: px = val
  }
  const fmt = (n) => Math.round(n * 10000) / 10000
  return [
    { unit: 'px', value: fmt(px) },
    { unit: 'rem', value: fmt(px / rootSize.value) },
    { unit: 'em', value: fmt(px / rootSize.value) },
    { unit: 'pt', value: fmt(px / 1.333333) },
    { unit: 'vw', value: fmt(px / (window.innerWidth / 100)) },
    { unit: 'vh', value: fmt(px / (window.innerHeight / 100)) },
    { unit: '%', value: fmt(px / basePx.value * 100) },
  ]
})

function convert() { /* computed 自动更新 */ }

function copy(text) {
  window.$he3?.copyText(text)
  window.$he3?.message.success('已复制 ' + text)
}
</script>

<style scoped>
.css-unit { display: flex; flex-direction: column; gap: 16px; }
.css-unit__row { display: flex; gap: 12px; }
.css-unit__field { flex: 1; }
.css-unit__field label { display: block; font-size: 12px; color: var(--text-tertiary); margin-bottom: 4px; }
.css-unit__field input { width: 100%; padding: 6px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); }
.css-unit__converter { display: flex; align-items: center; gap: 16px; flex-wrap: wrap; }
.css-unit__input-group { display: flex; gap: 0; }
.css-unit__input-group input { width: 100px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px 0 0 8px; background: var(--bg-surface); color: var(--text-primary); }
.css-unit__input-group select { padding: 8px 8px; border: 1px solid var(--border-color); border-left: none; border-radius: 0 8px 8px 0; background: var(--bg-surface); color: var(--text-primary); }
.css-unit__arrow { font-size: 20px; color: var(--text-tertiary); }
.css-unit__results { display: flex; gap: 8px; flex-wrap: wrap; }
.css-unit__result { display: flex; align-items: baseline; gap: 4px; padding: 6px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.css-unit__result:hover { border-color: var(--color-primary); }
.css-unit__num { font-size: 16px; font-weight: 700; color: var(--color-primary); }
.css-unit__unit { font-size: 12px; color: var(--text-tertiary); }
</style>
