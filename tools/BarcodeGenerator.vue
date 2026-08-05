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

<script setup>
import { ref, onMounted } from 'vue'
const text = ref('123456789012')
const type = ref('CODE128')
const types = [
  { id: 'CODE128', label: 'CODE128' },
  { id: 'CODE39', label: 'CODE39' },
  { id: 'EAN13', label: 'EAN13' },
  { id: 'UPC', label: 'UPC' }
]
const canvas = ref(null)
const generated = ref(false)

function generate() {
  if (!canvas.value || !text.value) return
  const ctx = canvas.value.getContext('2d')
  var data = text.value.split('').map(function (c) { return c.charCodeAt(0) })
  var barWidth = 2
  var width = data.length * 7 * barWidth + 40
  canvas.value.width = width
  canvas.value.height = 100
  ctx.fillStyle = '#fff'
  ctx.fillRect(0, 0, width, 100)
  var x = 20
  for (var i = 0; i < data.length; i++) {
    var byte = data[i]
    for (var bit = 0; bit < 7; bit++) {
      var isBlack = (byte >> bit) & 1
      ctx.fillStyle = isBlack ? '#000' : '#fff'
      for (var w = 0; w < barWidth; w++) {
        ctx.fillRect(x, 10, 1, 60)
        x++
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

function download() {
  if (!canvas.value) return
  var link = document.createElement('a')
  link.download = 'barcode-' + text.value + '.png'
  link.href = canvas.value.toDataURL()
  link.click()
}

onMounted(function () { generate() })
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
