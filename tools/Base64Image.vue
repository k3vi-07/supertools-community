<template>
  <h-single-layout>
    <div class="b64-img">
      <div class="b64-img__tabs">
        <button :class="{active: tab === 'encode'}" @click="tab = 'encode'">图片 → Base64</button>
        <button :class="{active: tab === 'decode'}" @click="tab = 'decode'">Base64 → 图片</button>
      </div>

      <!-- 图片 → Base64 -->
      <div v-if="tab === 'encode'" class="b64-img__encode">
        <div class="b64-img__drop" @drop.prevent="onDrop" @dragover.prevent>
          <input type="file" accept="image/*" @change="onFile" id="b64-file" hidden />
          <label for="b64-file" class="b64-img__drop-label">
            <span class="b64-img__drop-icon">📁</span>
            <span>点击选择或拖拽图片到这里</span>
            <span class="b64-img__drop-hint">支持 PNG / JPG / GIF / SVG / WebP</span>
          </label>
        </div>
        <div v-if="preview" class="b64-img__preview">
          <img :src="preview" class="b64-img__preview-img" />
          <div class="b64-img__preview-info">
            <span>{{ fileInfo.name }}</span>
            <span>{{ fileInfo.size }}</span>
            <span>{{ fileInfo.type }}</span>
          </div>
        </div>
        <div v-if="base64Result" class="b64-img__result">
          <textarea v-model="base64Result" class="b64-img__textarea" readonly></textarea>
          <button @click="copyB64">复制 Base64</button>
        </div>
      </div>

      <!-- Base64 → 图片 -->
      <div v-if="tab === 'decode'" class="b64-img__decode">
        <textarea v-model="b64Input" class="b64-img__textarea" placeholder="粘贴 Base64 字符串或 Data URL..." spellcheck="false"></textarea>
        <div v-if="decodedImg" class="b64-img__preview">
          <img :src="decodedImg" class="b64-img__preview-img" />
          <button @click="copyB64">复制 Data URL</button>
        </div>
        <div v-else-if="b64Input" class="b64-img__error">⚠️ 无效的图片 Base64 数据</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const tab = ref('encode')
const preview = ref('')
const fileInfo = ref({})
const base64Result = ref('')
const b64Input = ref('')

function formatSize(bytes) {
  if (bytes < 1024) return bytes + ' B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB'
  return (bytes / 1024 / 1024).toFixed(1) + ' MB'
}

function onFile(e) {
  const file = e.target.files[0]
  if (file) processFile(file)
}

function onDrop(e) {
  const file = e.dataTransfer.files[0]
  if (file && file.type.startsWith('image/')) processFile(file)
}

function processFile(file) {
  fileInfo.value = { name: file.name, size: formatSize(file.size), type: file.type }
  const reader = new FileReader()
  reader.onload = () => {
    preview.value = reader.result
    base64Result.value = reader.result
  }
  reader.readAsDataURL(file)
}

const decodedImg = computed(() => {
  if (!b64Input.value.trim()) return ''
  const raw = b64Input.value.trim()
  // 如果已经是 data URL
  if (raw.startsWith('data:image/')) return raw
  // 尝试添加前缀
  try {
    atob(raw.substring(0, 100))
    return 'data:image/png;base64,' + raw
  } catch {
    return ''
  }
})

function copyB64() {
  const text = tab.value === 'encode' ? base64Result.value : decodedImg.value
  window.$he3?.copyText(text)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.b64-img { display: flex; flex-direction: column; gap: 12px; }
.b64-img__tabs { display: flex; gap: 0; }
.b64-img__tabs button { padding: 8px 16px; border: 1px solid var(--border-color); background: var(--bg-surface); color: var(--text-secondary); cursor: pointer; }
.b64-img__tabs button:first-child { border-radius: 8px 0 0 8px; }
.b64-img__tabs button:last-child { border-radius: 0 8px 8px 0; border-left: none; }
.b64-img__tabs button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.b64-img__drop { border: 2px dashed var(--border-color); border-radius: 12px; padding: 24px; text-align: center; transition: border-color 0.2s; }
.b64-img__drop:hover { border-color: var(--color-primary); }
.b64-img__drop-label { cursor: pointer; display: flex; flex-direction: column; gap: 6px; align-items: center; }
.b64-img__drop-icon { font-size: 32px; }
.b64-img__drop-hint { font-size: 12px; color: var(--text-tertiary); }
.b64-img__preview { display: flex; align-items: flex-start; gap: 12px; }
.b64-img__preview-img { max-width: 200px; max-height: 200px; border-radius: 8px; border: 1px solid var(--border-color); }
.b64-img__preview-info { display: flex; flex-direction: column; gap: 4px; font-size: 13px; color: var(--text-secondary); }
.b64-img__result { display: flex; flex-direction: column; gap: 8px; }
.b64-img__textarea { width: 100%; min-height: 100px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 12px; resize: vertical; word-break: break-all; }
.b64-img__error { padding: 12px; border-radius: 8px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
.b64-img__encode button, .b64-img__decode button { padding: 8px 16px; border: none; border-radius: 8px; background: var(--color-primary); color: white; cursor: pointer; }
</style>
