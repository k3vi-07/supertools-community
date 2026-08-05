<template>
  <h-single-layout>
    <div class="qr-scanner">
      <p class="qr-scanner__hint">上传或拖入含二维码的图片进行识别</p>
      <label class="qr-scanner__dropzone">
        <input type="file" accept="image/*" @change="handleFile" hidden />
        <span>📷 选择图片</span>
      </label>
      <div v-if="result" class="qr-scanner__result">
        <div class="qr-scanner__header"><span>识别结果</span><button @click="copy">复制</button></div>
        <div class="qr-scanner__content selectable">{{ result }}</div>
      </div>
      <div v-if="error" class="qr-scanner__error">{{ error }}</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref } from 'vue'
const result = ref('')
const error = ref('')

async function handleFile(e) {
  const file = e.target.files?.[0]
  if (!file) return
  error.value = ''
  result.value = '识别中...'
  try {
    // 使用 BarcodeDetector API（Chromium 支持）
    const detector = new window.BarcodeDetector({ formats: ['qr_code'] })
    const img = await createImageBitmap(file)
    const codes = await detector.detect(img)
    if (codes.length > 0) {
      result.value = codes.map((c) => c.rawValue).join('\n')
    } else {
      result.value = ''
      error.value = '未检测到二维码'
    }
  } catch (err) {
    result.value = ''
    error.value = '识别失败: ' + err.message + '（请确保图片含清晰二维码）'
  }
}

function copy() {
  window.$he3?.copyText(result.value)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.qr-scanner { display: flex; flex-direction: column; align-items: center; gap: 16px; }
.qr-scanner__hint { color: var(--text-secondary); font-size: 14px; }
.qr-scanner__dropzone { display: flex; align-items: center; justify-content: center; width: 200px; height: 80px; border: 2px dashed var(--border-color); border-radius: 12px; cursor: pointer; transition: all 0.2s; }
.qr-scanner__dropzone:hover { border-color: var(--color-primary); background: color-mix(in srgb, var(--color-primary) 5%, transparent); }
.qr-scanner__dropzone span { color: var(--text-secondary); font-size: 14px; }
.qr-scanner__result { width: 100%; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.qr-scanner__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.qr-scanner__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.qr-scanner__content { padding: 12px; font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.qr-scanner__error { padding: 12px; border: 1px solid var(--color-error); border-radius: 8px; color: var(--color-error); font-size: 13px; }
</style>
