<template>
  <h-single-layout>
    <div class="cron-gen">
      <div class="cron-gen__preview">
        <code class="cron-gen__expr selectable">{{ cronExpression }}</code>
        <button class="cron-gen__copy" @click="copy">复制</button>
      </div>
      <p class="cron-gen__desc">{{ description }}</p>

      <div class="cron-gen__fields">
        <div class="cron-gen__field" v-for="f in fields" :key="f.key">
          <label>{{ f.label }}</label>
          <select v-model="f.mode">
            <option value="every">每个{{ f.unit }}</option>
            <option value="step">每隔</option>
            <option value="range">区间</option>
            <option value="specific">指定</option>
          </select>
          <div class="cron-gen__field-input">
            <template v-if="f.mode === 'step'">
              <span>每</span>
              <input type="number" v-model.number="f.step" min="1" :max="f.max" />
              <span>{{ f.unit }}</span>
            </template>
            <template v-else-if="f.mode === 'range'">
              <input type="number" v-model.number="f.rangeStart" :min="f.min" :max="f.max" />
              <span>-</span>
              <input type="number" v-model.number="f.rangeEnd" :min="f.min" :max="f.max" />
            </template>
            <template v-else-if="f.mode === 'specific'">
              <input type="text" v-model="f.specific" :placeholder="'如 ' + f.min + ',' + (f.min+1)" />
            </template>
          </div>
        </div>
      </div>

      <div class="cron-gen__examples">
        <h4>常用预设</h4>
        <div class="cron-gen__preset-list">
          <button v-for="p in presets" :key="p.expr" class="cron-gen__preset" @click="loadPreset(p)">
            <code>{{ p.expr }}</code>
            <span>{{ p.name }}</span>
          </button>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { reactive, computed } from 'vue'

const fields = reactive([
  { key: 'minute', label: '分钟', unit: '分钟', min: 0, max: 59, mode: 'every', step: 1, rangeStart: 0, rangeEnd: 30, specific: '0,15,30,45' },
  { key: 'hour', label: '小时', unit: '小时', min: 0, max: 23, mode: 'every', step: 2, rangeStart: 9, rangeEnd: 18, specific: '0,12' },
  { key: 'day', label: '日期', unit: '日', min: 1, max: 31, mode: 'every', step: 1, rangeStart: 1, rangeEnd: 15, specific: '1,15' },
  { key: 'month', label: '月份', unit: '月', min: 1, max: 12, mode: 'every', step: 3, rangeStart: 1, rangeEnd: 6, specific: '1,4,7,10' },
  { key: 'week', label: '星期', unit: '周', min: 0, max: 7, mode: 'every', step: 1, rangeStart: 1, rangeEnd: 5, specific: '1,3,5' }
])

function fieldExpr(f) {
  switch (f.mode) {
    case 'every': return '*'
    case 'step': return '*/' + (f.step || 1)
    case 'range': return f.rangeStart + '-' + f.rangeEnd
    case 'specific': return f.specific || '*'
    default: return '*'
  }
}

const cronExpression = computed(() => fields.map(fieldExpr).join(' '))

const description = computed(() => {
  const [m, h, d, mon, w] = fields
  const parts = []
  if (m.mode !== 'every') parts.push(fieldDesc(m, '分钟'))
  if (h.mode !== 'every') parts.push(fieldDesc(h, '小时'))
  if (d.mode !== 'every') parts.push(fieldDesc(d, '日'))
  if (mon.mode !== 'every') parts.push(fieldDesc(mon, '月'))
  if (w.mode !== 'every') parts.push(fieldDesc(w, '星期'))
  return parts.length > 0 ? parts.join('，') : '每分钟执行'
})

function fieldDesc(f, name) {
  switch (f.mode) {
    case 'step': return '每隔 ' + f.step + ' ' + name
    case 'range': return f.rangeStart + '-' + f.rangeEnd + ' ' + name
    case 'specific': return '指定 ' + name + ': ' + f.specific
    default: return '每' + name
  }
}

const presets = [
  { expr: '0 * * * *', name: '每小时', vals: ['specific', 0, 'every', 'every', 'every', 'every'] },
  { expr: '0 0 * * *', name: '每天午夜', vals: ['specific', 0, 'specific', 0, 'every', 'every', 'every'] },
  { expr: '0 9 * * 1-5', name: '工作日早9点', vals: ['specific', 0, 'specific', 9, 'every', 'every', 'range'] },
  { expr: '*/5 * * * *', name: '每5分钟', vals: ['step', 5, 'every', 'every', 'every', 'every', 'every'] },
  { expr: '0 0 1 * *', name: '每月1号', vals: ['specific', 0, 'specific', 0, 'specific', 1, 'every', 'every'] },
  { expr: '0 0 * * 0', name: '每周日', vals: ['specific', 0, 'specific', 0, 'every', 'every', 'specific'] }
]

function loadPreset(p) {
  // 简化：直接按表达式解析
  const parts = p.expr.split(' ')
  const modes = ['every', 'step', 'range', 'specific']
  fields.forEach((f, i) => {
    const val = parts[i]
    if (val === '*') { f.mode = 'every' }
    else if (val.startsWith('*/')) { f.mode = 'step'; f.step = parseInt(val.slice(2)) }
    else if (val.includes('-')) { f.mode = 'range'; const [a, b] = val.split('-'); f.rangeStart = parseInt(a); f.rangeEnd = parseInt(b) }
    else { f.mode = 'specific'; f.specific = val }
  })
}

function copy() {
  navigator.clipboard?.writeText(cronExpression.value)
}
</script>

<style scoped>
.cron-gen { display: flex; flex-direction: column; gap: 16px; }
.cron-gen__preview { display: flex; align-items: center; gap: 12px; }
.cron-gen__expr { font-size: 22px; font-weight: 700; color: var(--color-primary); font-family: monospace; flex: 1; }
.cron-gen__copy { padding: 6px 16px; border: 1px solid var(--color-primary); border-radius: 6px; background: transparent; color: var(--color-primary); cursor: pointer; font-size: 13px; }
.cron-gen__copy:hover { background: var(--color-primary); color: white; }
.cron-gen__desc { font-size: 14px; color: var(--text-secondary); }
.cron-gen__fields { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 12px; }
.cron-gen__field { display: flex; flex-direction: column; gap: 6px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.cron-gen__field label { font-size: 12px; color: var(--text-tertiary); }
.cron-gen__field select { padding: 6px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-primary); font-size: 13px; }
.cron-gen__field-input { display: flex; align-items: center; gap: 6px; font-size: 12px; color: var(--text-secondary); }
.cron-gen__field-input input { width: 50px; padding: 4px 6px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-primary); font-size: 12px; }
.cron-gen__examples h4 { font-size: 13px; color: var(--text-secondary); margin-bottom: 8px; }
.cron-gen__preset-list { display: flex; flex-wrap: wrap; gap: 8px; }
.cron-gen__preset { display: flex; flex-direction: column; align-items: center; gap: 2px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; }
.cron-gen__preset:hover { border-color: var(--color-primary); }
.cron-gen__preset code { font-size: 12px; color: var(--color-primary); }
.cron-gen__preset span { font-size: 11px; color: var(--text-tertiary); }
</style>
