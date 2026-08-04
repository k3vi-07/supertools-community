<template>
  <h-single-layout>
    <div class="chmod">
      <div class="chmod__permissions">
        <div v-for="(perm, idx) in perms" :key="idx" class="chmod__group">
          <span class="chmod__group-name">{{ ['所有者', '组', '其他'][idx] }}</span>
          <div class="chmod__checks">
            <label v-for="b in bits" :key="b.value"><input type="checkbox" :checked="perm & b.value" @change="toggle(idx, b.value)" /> {{ b.label }}</label>
          </div>
        </div>
      </div>
      <div class="chmod__results">
        <div class="chmod__result-card"><span class="chmod__result-label">数字</span><code class="chmod__code">{{ octal }}</code></div>
        <div class="chmod__result-card"><span class="chmod__result-label">符号</span><code class="chmod__code">{{ symbolic }}</code></div>
        <div class="chmod__result-card"><span class="chmod__result-label">命令</span><code class="chmod__code selectable">chmod {{ octal }} file</code></div>
      </div>
      <div class="chmod__desc">{{ description }}</div>
    </div>
  </h-single-layout>
</template>

<script setup lang="vue">
</script>

<script lang="ts">
import { ref, computed } from 'vue'
const bits = [
  { value: 4, label: '读 (r)' },
  { value: 2, label: '写 (w)' },
  { value: 1, label: '执行 (x)' }
]
const perms = ref([7, 5, 5])

function toggle(idx: number, bit: number): void {
  perms.value[idx] ^= bit
}
const octal = computed(() => perms.value.map((p) => p.toString(8)).join(''))
const symbolic = computed(() => perms.value.map((p) => (p & 4 ? 'r' : '-') + (p & 2 ? 'w' : '-') + (p & 1 ? 'x' : '-')).join(''))
const description = computed(() => {
  const labels = ['所有者', '组', '其他']
  return perms.value.map((p, i) => {
    const parts: string[] = []
    if (p & 4) parts.push('读')
    if (p & 2) parts.push('写')
    if (p & 1) parts.push('执行')
    return `${labels[i]}: ${parts.join(' + ') || '无权限'}`
  }).join('；')
})
</script>

<style scoped>
.chmod { display: flex; flex-direction: column; gap: 16px; }
.chmod__permissions { display: flex; gap: 16px; }
.chmod__group { flex: 1; display: flex; flex-direction: column; gap: 8px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.chmod__group-name { font-size: 13px; font-weight: 600; color: var(--text-primary); }
.chmod__checks { display: flex; flex-direction: column; gap: 6px; }
.chmod__checks label { display: flex; align-items: center; gap: 6px; font-size: 12px; color: var(--text-secondary); cursor: pointer; }
.chmod__results { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.chmod__result-card { display: flex; flex-direction: column; gap: 4px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.chmod__result-label { font-size: 11px; color: var(--text-tertiary); }
.chmod__code { font-family: monospace; font-size: 16px; font-weight: 700; color: var(--color-primary); }
.chmod__desc { padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-base); font-size: 13px; color: var(--text-secondary); }
</style>
