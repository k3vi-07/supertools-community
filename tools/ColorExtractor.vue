<template>
  <h-single-layout>
    <div class="color-ext">
      <textarea v-model="input" class="color-ext__input" placeholder="粘贴 CSS / HTML 代码，自动提取颜色..." spellcheck="false"></textarea>
      <div v-if="colors.length" class="color-ext__stats">
        <span>找到 {{ colors.length }} 种颜色</span>
        <button @click="copyAll">复制全部</button>
      </div>
      <div class="color-ext__grid">
        <div v-for="c in uniqueColors" :key="c" class="color-ext__item" @click="copy(c)">
          <div class="color-ext__swatch" :style="{ background: c }"></div>
          <div class="color-ext__info">
            <div class="color-ext__hex">{{ c }}</div>
            <div class="color-ext__rgb">{{ toRgb(c) }}</div>
          </div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref(`body {
  background: #1a1a2e;
  color: rgba(255, 255, 255, 0.9);
  --primary: #6366f1;
  --success: #22c55e;
  --danger: #ef4444;
}
.btn { border: 1px solid hsl(210, 40%, 50%); }`)

const colorRegex = /(#(?:[0-9a-fA-F]{3}){1,2}\b|rgba?\([^)]+\)|hsla?\([^)]+\))/g

const colors = computed(() => {
  const found = input.value.match(colorRegex)
  return found || []
})

const uniqueColors = computed(() => {
  return [...new Set(colors.value)]
})

function toRgb(color) {
  if (color.startsWith('#')) {
    const hex = color.substring(1)
    const full = hex.length === 3 ? hex.split('').map(c => c + c).join('') : hex
    const r = parseInt(full.substring(0, 2), 16)
    const g = parseInt(full.substring(2, 4), 16)
    const b = parseInt(full.substring(4, 6), 16)
    return `rgb(${r}, ${g}, ${b})`
  }
  return color
}

function copy(c) {
  window.$he3?.copyText(c)
  window.$he3?.message.success('已复制 ' + c)
}

function copyAll() {
  window.$he3?.copyText(uniqueColors.value.join('\n'))
  window.$he3?.message.success('已复制 ' + uniqueColors.value.length + ' 个颜色')
}
</script>

<style scoped>
.color-ext { display: flex; flex-direction: column; gap: 12px; }
.color-ext__input { width: 100%; min-height: 120px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.color-ext__stats { display: flex; align-items: center; justify-content: space-between; font-size: 13px; color: var(--text-secondary); }
.color-ext__stats button { padding: 4px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; font-size: 12px; }
.color-ext__stats button:hover { border-color: var(--color-primary); color: var(--color-primary); }
.color-ext__grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 8px; }
.color-ext__item { display: flex; align-items: center; gap: 10px; padding: 8px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.color-ext__item:hover { border-color: var(--color-primary); transform: translateY(-2px); }
.color-ext__swatch { width: 40px; height: 40px; border-radius: 8px; border: 1px solid var(--border-color); flex-shrink: 0; }
.color-ext__info { display: flex; flex-direction: column; gap: 2px; min-width: 0; }
.color-ext__hex { font-size: 13px; font-weight: 600; color: var(--text-primary); }
.color-ext__rgb { font-size: 11px; color: var(--text-tertiary); }
</style>
