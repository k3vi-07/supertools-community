<template>
  <h-single-layout>
    <div class="mime-types">
      <input v-model="search" placeholder="搜索扩展名或 MIME 类型... (如 .json, image/, pdf)" />
      <div class="mime-types__list">
        <div v-for="m in filtered" :key="m.ext" class="mime-types__item" @click="copy(m.mime)">
          <span class="mime-types__ext">{{ m.ext }}</span>
          <span class="mime-types__mime selectable">{{ m.mime }}</span>
          <span class="mime-types__type" :class="m.type">{{ typeLabel(m.type) }}</span>
        </div>
      </div>
      <div v-if="filtered.length === 0" class="mime-types__empty">未找到匹配的类型</div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const search = ref('')

const types = [
  { ext: '.html', mime: 'text/html', type: 'text' },
  { ext: '.css', mime: 'text/css', type: 'text' },
  { ext: '.js', mime: 'text/javascript', type: 'text' },
  { ext: '.ts', mime: 'text/plain', type: 'text' },
  { ext: '.json', mime: 'application/json', type: 'app' },
  { ext: '.xml', mime: 'application/xml', type: 'app' },
  { ext: '.csv', mime: 'text/csv', type: 'text' },
  { ext: '.txt', mime: 'text/plain', type: 'text' },
  { ext: '.md', mime: 'text/markdown', type: 'text' },
  { ext: '.yaml', mime: 'text/yaml', type: 'text' },
  { ext: '.pdf', mime: 'application/pdf', type: 'app' },
  { ext: '.zip', mime: 'application/zip', type: 'app' },
  { ext: '.gz', mime: 'application/gzip', type: 'app' },
  { ext: '.tar', mime: 'application/x-tar', type: 'app' },
  { ext: '.png', mime: 'image/png', type: 'image' },
  { ext: '.jpg', mime: 'image/jpeg', type: 'image' },
  { ext: '.gif', mime: 'image/gif', type: 'image' },
  { ext: '.svg', mime: 'image/svg+xml', type: 'image' },
  { ext: '.webp', mime: 'image/webp', type: 'image' },
  { ext: '.ico', mime: 'image/x-icon', type: 'image' },
  { ext: '.mp4', mime: 'video/mp4', type: 'video' },
  { ext: '.webm', mime: 'video/webm', type: 'video' },
  { ext: '.avi', mime: 'video/x-msvideo', type: 'video' },
  { ext: '.mp3', mime: 'audio/mpeg', type: 'audio' },
  { ext: '.wav', mime: 'audio/wav', type: 'audio' },
  { ext: '.ogg', mime: 'audio/ogg', type: 'audio' },
  { ext: '.flac', mime: 'audio/flac', type: 'audio' },
  { ext: '.woff', mime: 'font/woff', type: 'font' },
  { ext: '.woff2', mime: 'font/woff2', type: 'font' },
  { ext: '.ttf', mime: 'font/ttf', type: 'font' },
  { ext: '.otf', mime: 'font/otf', type: 'font' },
  { ext: '.doc', mime: 'application/msword', type: 'app' },
  { ext: '.docx', mime: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', type: 'app' },
  { ext: '.xls', mime: 'application/vnd.ms-excel', type: 'app' },
  { ext: '.xlsx', mime: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', type: 'app' },
  { ext: '.ppt', mime: 'application/vnd.ms-powerpoint', type: 'app' },
  { ext: '.pptx', mime: 'application/vnd.openxmlformats-officedocument.presentationml.presentation', type: 'app' },
  { ext: '.wasm', mime: 'application/wasm', type: 'app' },
  { ext: '.jar', mime: 'application/java-archive', type: 'app' },
  { ext: '.dmg', mime: 'application/x-apple-diskimage', type: 'app' }
]

const filtered = computed(() => {
  if (!search.value.trim()) return types
  const q = search.value.toLowerCase().replace(/^\./, '')
  return types.filter(m =>
    m.ext.includes(q) || m.mime.includes(q) || m.type.includes(q)
  )
})

function typeLabel(t) {
  const labels = { text: '文本', app: '应用', image: '图片', video: '视频', audio: '音频', font: '字体' }
  return labels[t] || t
}

function copy(text) {
  navigator.clipboard?.writeText(text)
}
</script>

<style scoped>
.mime-types { display: flex; flex-direction: column; gap: 12px; }
.mime-types input { padding: 10px 14px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.mime-types__list { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 6px; }
.mime-types__item { display: flex; align-items: center; gap: 10px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.mime-types__item:hover { border-color: var(--color-primary); }
.mime-types__ext { font-family: monospace; font-size: 13px; font-weight: 600; color: var(--color-primary); min-width: 60px; }
.mime-types__mime { flex: 1; font-family: monospace; font-size: 12px; color: var(--text-secondary); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.mime-types__type { font-size: 10px; padding: 2px 8px; border-radius: 10px; flex-shrink: 0; }
.mime-types__type.text { background: rgba(59,130,246,0.15); color: #3b82f6; }
.mime-types__type.app { background: rgba(168,85,247,0.15); color: #a855f7; }
.mime-types__type.image { background: rgba(34,197,94,0.15); color: #22c55e; }
.mime-types__type.video { background: rgba(239,68,68,0.15); color: #ef4444; }
.mime-types__type.audio { background: rgba(245,158,11,0.15); color: #f59e0b; }
.mime-types__type.font { background: rgba(99,102,241,0.15); color: #6366f1; }
.mime-types__empty { text-align: center; padding: 32px; color: var(--text-tertiary); font-size: 14px; }
</style>
