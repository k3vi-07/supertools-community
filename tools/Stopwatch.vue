<template>
  <h-single-layout>
    <div class="stopwatch">
      <div class="stopwatch__display">{{ formatted }}</div>
      <div class="stopwatch__controls">
        <button class="stopwatch__btn stopwatch__btn--primary" @click="toggle">{{ running ? '暂停' : '开始' }}</button>
        <button class="stopwatch__btn" @click="lap" :disabled="!running">记圈</button>
        <button class="stopwatch__btn stopwatch__btn--danger" @click="reset">重置</button>
      </div>
      <div v-if="laps.length" class="stopwatch__laps">
        <div v-for="(lap, i) in laps" :key="i" class="stopwatch__lap">
          <span>圈 {{ i + 1 }}</span><code>{{ formatTime(lap) }}</code>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue'
const elapsed = ref(0)
const running = ref(false)
const laps = ref<number[]>([])
let start = 0
let raf = 0

function tick(): void { elapsed.value = Date.now() - start; raf = requestAnimationFrame(tick) }
function toggle(): void {
  if (running.value) { running.value = false; cancelAnimationFrame(raf) }
  else { running.value = true; start = Date.now() - elapsed.value; raf = requestAnimationFrame(tick) }
}
function lap(): void { laps.value.unshift(elapsed.value) }
function reset(): void { running.value = false; cancelAnimationFrame(raf); elapsed.value = 0; laps.value = [] }
const formatted = computed(() => formatTime(elapsed.value))
function formatTime(ms: number): string {
  const total = Math.floor(ms / 10)
  const min = Math.floor(total / 6000)
  const sec = Math.floor((total % 6000) / 100)
  const cs = total % 100
  return `${String(min).padStart(2, '0')}:${String(sec).padStart(2, '0')}.${String(cs).padStart(2, '0')}`
}
onUnmounted(() => cancelAnimationFrame(raf))
</script>

<style scoped>
.stopwatch { display: flex; flex-direction: column; gap: 20px; align-items: center; }
.stopwatch__display { font-size: 56px; font-weight: 800; color: var(--color-primary); font-family: monospace; padding: 20px; }
.stopwatch__controls { display: flex; gap: 12px; }
.stopwatch__btn { padding: 10px 24px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; cursor: pointer; transition: all 0.15s; }
.stopwatch__btn:disabled { opacity: 0.4; cursor: not-allowed; }
.stopwatch__btn--primary { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.stopwatch__btn--danger { color: var(--color-error); border-color: var(--color-error); }
.stopwatch__laps { width: 100%; max-height: 200px; overflow-y: auto; }
.stopwatch__lap { display: flex; justify-content: space-between; padding: 8px 16px; border-bottom: 1px solid var(--border-color-light); font-size: 13px; }
.stopwatch__lap code { font-family: monospace; color: var(--color-primary); }
</style>
