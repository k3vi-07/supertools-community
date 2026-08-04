<template>
  <h-single-layout>
    <div class="unit-converter">
      <div class="unit-converter__row">
        <div class="unit-converter__field">
          <label>数值</label>
          <input type="number" v-model.number="value" step="any" />
        </div>
        <div class="unit-converter__field">
          <label>从</label>
          <select v-model="fromUnit"><option v-for="u in currentUnits" :key="u.id" :value="u.id">{{ u.label }}</option></select>
        </div>
        <div class="unit-converter__arrow">→</div>
        <div class="unit-converter__field">
          <label>到</label>
          <select v-model="toUnit"><option v-for="u in currentUnits" :key="u.id" :value="u.id">{{ u.label }}</option></select>
        </div>
      </div>
      <div class="unit-converter__types">
        <button v-for="t in types" :key="t.id" :class="{active: type===t.id}" @click="switchType(t.id)">{{ t.label }}</button>
      </div>
      <div class="unit-converter__result">{{ result }}</div>
      <div class="unit-converter__all">
        <div v-for="u in currentUnits" :key="u.id" class="unit-converter__item">
          <span class="unit-converter__unit">{{ u.label }}</span>
          <span class="unit-converter__val">{{ convertTo(value, u.id).toFixed(6).replace(/\.?0+$/, '') }}</span>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const types = [
  { id: 'length', label: '长度' },
  { id: 'weight', label: '重量' },
  { id: 'temperature', label: '温度' },
  { id: 'area', label: '面积' },
  { id: 'speed', label: '速度' },
  { id: 'data', label: '数据' }
]

const allUnits: Record<string, { id: string; label: string; factor: number }[]> = {
  length: [
    { id: 'mm', label: '毫米', factor: 0.001 },
    { id: 'cm', label: '厘米', factor: 0.01 },
    { id: 'm', label: '米', factor: 1 },
    { id: 'km', label: '千米', factor: 1000 },
    { id: 'in', label: '英寸', factor: 0.0254 },
    { id: 'ft', label: '英尺', factor: 0.3048 },
    { id: 'mi', label: '英里', factor: 1609.344 }
  ],
  weight: [
    { id: 'mg', label: '毫克', factor: 0.001 },
    { id: 'g', label: '克', factor: 1 },
    { id: 'kg', label: '千克', factor: 1000 },
    { id: 't', label: '吨', factor: 1000000 },
    { id: 'oz', label: '盎司', factor: 28.3495 },
    { id: 'lb', label: '磅', factor: 453.592 }
  ],
  temperature: [
    { id: 'c', label: '摄氏度 °C', factor: 1 },
    { id: 'f', label: '华氏度 °F', factor: 1 },
    { id: 'k', label: '开尔文 K', factor: 1 }
  ],
  area: [
    { id: 'cm2', label: '平方厘米', factor: 0.0001 },
    { id: 'm2', label: '平方米', factor: 1 },
    { id: 'km2', label: '平方千米', factor: 1000000 },
    { id: 'ha', label: '公顷', factor: 10000 },
    { id: 'acre', label: '英亩', factor: 4046.86 },
    { id: 'ft2', label: '平方英尺', factor: 0.092903 }
  ],
  speed: [
    { id: 'ms', label: '米/秒', factor: 1 },
    { id: 'kmh', label: '千米/时', factor: 0.277778 },
    { id: 'mph', label: '英里/时', factor: 0.44704 },
    { id: 'knot', label: '节', factor: 0.514444 },
    { id: 'mach', label: '马赫', factor: 343 }
  ],
  data: [
    { id: 'b', label: 'Byte', factor: 1 },
    { id: 'kb', label: 'KB', factor: 1024 },
    { id: 'mb', label: 'MB', factor: 1048576 },
    { id: 'gb', label: 'GB', factor: 1073741824 },
    { id: 'tb', label: 'TB', factor: 1099511627776 }
  ]
}

const type = ref('length')
const value = ref(1)
const fromUnit = ref('m')
const toUnit = ref('ft')

const currentUnits = computed(() => allUnits[type.value] || [])

function switchType(t: string): void {
  type.value = t
  const units = allUnits[t]
  if (units) { fromUnit.value = units[0].id; toUnit.value = units[1]?.id || units[0].id }
}

function convertTo(val: number, target: string): number {
  if (type.value === 'temperature') return convertTemp(val, fromUnit.value, target)
  const fromFactor = currentUnits.value.find((u) => u.id === fromUnit.value)?.factor || 1
  const toFactor = currentUnits.value.find((u) => u.id === target)?.factor || 1
  return (val * fromFactor) / toFactor
}

function convertTemp(val: number, from: string, to: string): number {
  let c: number
  if (from === 'c') c = val
  else if (from === 'f') c = (val - 32) * 5 / 9
  else c = val - 273.15
  if (to === 'c') return c
  if (to === 'f') return c * 9 / 5 + 32
  return c + 273.15
}

const result = computed(() => {
  const converted = convertTo(value.value, toUnit.value)
  const fromLabel = currentUnits.value.find((u) => u.id === fromUnit.value)?.label || ''
  const toLabel = currentUnits.value.find((u) => u.id === toUnit.value)?.label || ''
  return `${value.value} ${fromLabel} = ${converted.toFixed(6).replace(/\.?0+$/, '')} ${toLabel}`
})
</script>

<style scoped>
.unit-converter { display: flex; flex-direction: column; gap: 16px; }
.unit-converter__row { display: flex; align-items: flex-end; gap: 12px; }
.unit-converter__field { display: flex; flex-direction: column; gap: 4px; flex: 1; }
.unit-converter__field label { font-size: 12px; color: var(--text-secondary); }
.unit-converter__field input, .unit-converter__field select { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.unit-converter__arrow { font-size: 20px; color: var(--text-tertiary); padding-bottom: 8px; }
.unit-converter__types { display: flex; gap: 4px; flex-wrap: wrap; }
.unit-converter__types button { padding: 4px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.unit-converter__types button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.unit-converter__result { padding: 16px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-size: 18px; font-weight: 600; color: var(--color-primary); text-align: center; }
.unit-converter__all { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 8px; }
.unit-converter__item { display: flex; justify-content: space-between; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; }
.unit-converter__unit { color: var(--text-tertiary); }
.unit-converter__val { font-family: monospace; color: var(--text-primary); }
</style>
