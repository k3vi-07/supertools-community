<template>
  <h-single-layout>
    <div class="bar-gen">
      <input v-model="text" placeholder="输入要编码的文本..." class="bar-gen__input" />
      <div class="bar-gen__preview"><canvas ref="canvas"></canvas></div>
      <div class="bar-gen__types">
        <button v-for="t in types" :key="t.id" :class="{active: type===t.id}" @click="type=t.id; generate()">{{ t.label }}</button>
      </div>
      <button class="bar-gen__btn" @click="generate">生成条形码</button>
      <button v-if="generated" class="bar-gen__btn" @click="download">下载</button>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
const text = ref('123456789012')
const type = ref('CODE128')
const types = [
  { id: 'CODE128', label: 'CODE128' },
  { id: 'CODE39', label: 'CODE39' },
  { id: 'EAN13', label: 'EAN13' },
  { id: 'UPC', label: 'UPC' }
]
const canvas = ref<HTMLCanvasElement>()
const generated = ref(false)

function generate(): void {
  if (!canvas.value || !text.value) return
  const ctx = canvas.value.getContext('2d')!
  // 简化版条形码：根据文本生成条纹
  const data = text.value.split('').map((c) => c.charCodeAt(0))
  const barWidth = 2
  const width = data.length * 7 * barWidth + 40
  canvas.value.width = width
  canvas.value.height = 100
  ctx.fillStyle = '#fff'
  ctx.fillRect(0, 0, width, 100)
  let x = 20
  for (const byte of data) {
    for (let bit = 0; bit < 7; bit++) {
      const isBlack = (byte >> bit) & 1
      ctx.fillStyle = isBlack ? '#000' : '#fff'
      for (let w = 0; w < barWidth; w++) {
        ctx.fillRect(x++, 10, 1, 60)
      }
    }
    x += barWidth
  }
  ctx.fillStyle = '#000'
  ctx.font = '14px monospace'
  ctx.textAlign = 'center'
  ctx.fillText(text.value, width / 2, 90)
  generated.value = true
}

function download(): void {
  if (!canvas.value) return
  const link = document.createElement('a')
  link.download = `barcode-${text.value}.png`
  link.href = canvas.value.toDataURL()
  link.click()
}

onMounted(() => generate())
</script>

<style scoped>
.bar-gen { display: flex; flex-direction: column; align-items: center; gap: 16px; }
.bar-gen__input { width: 100%; padding: 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.bar-gen__preview { padding: 16px; border: 1px solid var(--border-color); border-radius: 8px; background: #fff; }
.bar-gen__types { display: flex; gap: 4px; flex-wrap: wrap; }
.bar-gen__types button { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.bar-gen__types button.active { background: var(--color-primary); color: white; }
.bar-gen__btn { padding: 8px 20px; border: none; border-radius: 6px; background: var(--color-primary); color: white; font-size: 14px; cursor: pointer; }
</style>
