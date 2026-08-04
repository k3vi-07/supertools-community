<template>
  <h-single-layout>
    <div class="counter">
      <div class="counter__input"><label>目标日期时间</label><input type="datetime-local" v-model="targetDate" /></div>
      <div class="counter__display">
        <div v-for="unit in units" :key="unit.label" class="counter__unit">
          <div class="counter__num">{{ unit.value }}</div>
          <div class="counter__label">{{ unit.label }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
const defaultDate = new Date(Date.now() + 7 * 24 * 3600 * 1000)
const targetDate = ref(new Date(defaultDate.getTime() - defaultDate.getTimezoneOffset() * 60000).toISOString().slice(0, 16))
const now = ref(Date.now())
let timer: ReturnType<typeof setInterval> | null = null

const units = computed(() => {
  const diff = new Date(targetDate.value).getTime() - now.value
  const isPast = diff < 0
  const abs = Math.abs(diff)
  const days = Math.floor(abs / 86400000)
  const hours = Math.floor((abs % 86400000) / 3600000)
  const mins = Math.floor((abs % 3600000) / 60000)
  const secs = Math.floor((abs % 60000) / 1000)
  return [
    { label: isPast ? '天前' : '天', value: String(days).padStart(2, '0') },
    { label: isPast ? '小时前' : '时', value: String(hours).padStart(2, '0') },
    { label: isPast ? '分钟前' : '分', value: String(mins).padStart(2, '0') },
    { label: isPast ? '秒前' : '秒', value: String(secs).padStart(2, '0') }
  ]
})

onMounted(() => { timer = setInterval(() => { now.value = Date.now() }, 1000) })
onUnmounted(() => { if (timer) clearInterval(timer) })
</script>

<style scoped>
.counter { display: flex; flex-direction: column; gap: 20px; }
.counter__input { display: flex; flex-direction: column; gap: 4px; }
.counter__input label { font-size: 12px; color: var(--text-secondary); }
.counter__input input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.counter__display { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; }
.counter__unit { display: flex; flex-direction: column; align-items: center; gap: 4px; padding: 20px 8px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.counter__num { font-size: 36px; font-weight: 800; color: var(--color-primary); font-family: monospace; }
.counter__label { font-size: 12px; color: var(--text-tertiary); }
</style>
