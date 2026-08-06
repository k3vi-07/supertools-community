<template>
  <h-single-layout>
    <div class="cron-gen">
      <div class="cron-gen__presets">
        <button v-for="p in presets" :key="p.label" class="cron-gen__preset" @click="applyPreset(p)">{{ p.label }}</button>
      </div>
      <div class="cron-gen__fields">
        <div v-for="(f, i) in fields" :key="f.name" class="cron-gen__field">
          <label>{{ f.name }}</label>
          <input v-model="parts[i]" @input="buildExpression" :placeholder="f.hint" />
        </div>
      </div>
      <div class="cron-gen__result">
        <div class="cron-gen__expr-row">
          <code class="cron-gen__expr">{{ expression }}</code>
          <button @click="copy">复制</button>
        </div>
        <div class="cron-gen__desc">{{ description }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const fields = [
  { name: '秒', hint: '0-59 / *' },
  { name: '分', hint: '0-59 / *' },
  { name: '时', hint: '0-23 / *' },
  { name: '日', hint: '1-31 / *' },
  { name: '月', hint: '1-12 / *' },
  { name: '周', hint: '0-6 / *' },
]

const parts = ref(['0', '*/5', '*', '*', '*', '*'])

const expression = computed(() => parts.value.join(' '))

const description = computed(() => {
  const [sec, min, hour, day, month, week] = parts.value
  const desc = []
  if (min.includes('*/')) desc.push(`每 ${min.split('*/')[1]} 分钟`)
  else if (min === '*') desc.push('每分钟')
  else desc.push(`第 ${min} 分`)

  if (hour !== '*') desc[0] = `每天 ${hour}:${min.padStart(2, '0')}`
  if (day !== '*' && day !== '?') desc.push(`每月 ${day} 号`)
  if (week !== '*' && week !== '?') {
    const names = ['日', '一', '二', '三', '四', '五', '六']
    desc.push(`每周${names[parseInt(week)] || week}`)
  }
  if (month !== '*') desc.push(`${month} 月`)

  return desc.join(' · ')
})

const presets = [
  { label: '每分钟', vals: ['0', '*', '*', '*', '*', '?'] },
  { label: '每5分钟', vals: ['0', '*/5', '*', '*', '*', '?'] },
  { label: '每小时', vals: ['0', '0', '*', '*', '*', '?'] },
  { label: '每天0点', vals: ['0', '0', '0', '*', '*', '?'] },
  { label: '每周一', vals: ['0', '0', '0', '?', '*', '1'] },
  { label: '每月1号', vals: ['0', '0', '0', '1', '*', '?'] },
  { label: '工作日', vals: ['0', '0', '9', '?', '*', '1-5'] },
  { label: '每天9点', vals: ['0', '0', '9', '*', '*', '?'] },
  { label: '每天18点', vals: ['0', '0', '18', '*', '*', '?'] },
]

function applyPreset(p) {
  parts.value = [...p.vals]
}

function buildExpression() { /* computed 自动更新 */ }

function copy() {
  window.$he3?.copyText(expression.value)
  window.$he3?.message.success('已复制 ' + expression.value)
}
</script>

<style scoped>
.cron-gen { display: flex; flex-direction: column; gap: 12px; }
.cron-gen__presets { display: flex; gap: 6px; flex-wrap: wrap; }
.cron-gen__preset { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; font-size: 12px; }
.cron-gen__preset:hover { border-color: var(--color-primary); color: var(--color-primary); }
.cron-gen__fields { display: grid; grid-template-columns: repeat(6, 1fr); gap: 8px; }
.cron-gen__field { display: flex; flex-direction: column; gap: 4px; }
.cron-gen__field label { font-size: 11px; color: var(--text-tertiary); text-align: center; }
.cron-gen__field input { width: 100%; padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); text-align: center; font-family: monospace; }
.cron-gen__result { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.cron-gen__expr-row { display: flex; align-items: center; gap: 12px; }
.cron-gen__expr { flex: 1; font-size: 18px; font-weight: 700; color: var(--color-primary); word-break: break-all; }
.cron-gen__expr-row button { padding: 6px 14px; border: none; border-radius: 6px; background: var(--color-primary); color: white; cursor: pointer; }
.cron-gen__desc { margin-top: 8px; font-size: 13px; color: var(--text-secondary); }
</style>
