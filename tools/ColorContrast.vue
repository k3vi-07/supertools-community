<template>
  <h-single-layout>
    <div class="color-contrast">
      <div class="color-contrast__colors">
        <div class="color-contrast__picker"><label>前景色</label><input type="color" v-model="fg" /></div>
        <div class="color-contrast__picker"><label>背景色</label><input type="color" v-model="bg" /></div>
        <div class="color-contrast__preview" :style="{ color: fg, background: bg }">
          <div class="color-contrast__preview-text">Aa Bb 文字示例 123</div>
          <div class="color-contrast__small" :style="{ fontSize: '12px' }">小号文字 Small Text</div>
        </div>
      </div>
      <div class="color-contrast__results">
        <div class="color-contrast__card" :class="ratio >= 4.5 ? 'pass' : 'fail'">
          <span>对比度</span><strong>{{ ratio.toFixed(2) }}:1</strong>
          <span class="color-contrast__badge">{{ ratio >= 4.5 ? '✓ 达标' : '✗ 不达标' }}</span>
        </div>
        <div class="color-contrast__card">
          <span>AA 标准 (4.5:1)</span><strong>{{ ratio >= 4.5 ? '✅ 通过' : '❌ 未通过' }}</strong>
        </div>
        <div class="color-contrast__card">
          <span>AAA 标准 (7:1)</span><strong>{{ ratio >= 7 ? '✅ 通过' : '❌ 未通过' }}</strong>
        </div>
        <div class="color-contrast__card">
          <span>大文字 AA (3:1)</span><strong>{{ ratio >= 3 ? '✅ 通过' : '❌ 未通过' }}</strong>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const fg = ref('#000000')
const bg = ref('#ffffff')

const ratio = computed(() => {
  const l1 = getLuminance(fg.value)
  const l2 = getLuminance(bg.value)
  const lighter = Math.max(l1, l2)
  const darker = Math.min(l1, l2)
  return (lighter + 0.05) / (darker + 0.05)
})

function getLuminance(hex) {
  const r = parseInt(hex.slice(1, 3), 16) / 255
  const g = parseInt(hex.slice(3, 5), 16) / 255
  const b = parseInt(hex.slice(5, 7), 16) / 255
  return 0.2126 * linearize(r) + 0.7152 * linearize(g) + 0.0722 * linearize(b)
}
function linearize(c) {
  return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
}
</script>

<style scoped>
.color-contrast { display: flex; flex-direction: column; gap: 16px; }
.color-contrast__colors { display: flex; gap: 16px; }
.color-contrast__picker { display: flex; flex-direction: column; gap: 4px; }
.color-contrast__picker label { font-size: 12px; color: var(--text-secondary); }
.color-contrast__picker input { width: 60px; height: 40px; border: 1px solid var(--border-color); border-radius: 6px; cursor: pointer; }
.color-contrast__preview { flex: 1; padding: 20px; border-radius: 8px; border: 1px solid var(--border-color); display: flex; flex-direction: column; gap: 8px; }
.color-contrast__preview-text { font-size: 20px; font-weight: 600; }
.color-contrast__results { display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
.color-contrast__card { display: flex; flex-direction: column; gap: 4px; padding: 14px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.color-contrast__card span { font-size: 12px; color: var(--text-tertiary); }
.color-contrast__card strong { font-size: 16px; color: var(--text-primary); }
.color-contrast__badge { font-size: 12px; font-weight: 600; }
.color-contrast__card.pass { border-color: var(--color-success); }
.color-contrast__card.fail { border-color: var(--color-error); }
</style>
