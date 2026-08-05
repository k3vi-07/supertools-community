<template>
  <h-single-layout>
    <div class="csp-gen">
      <div class="csp-gen__policies">
        <div v-for="p in policies" :key="p.directive" class="csp-gen__policy">
          <label class="csp-gen__policy-label">
            <input type="checkbox" v-model="p.enabled" />
            <code>{{ p.directive }}</code>
          </label>
          <span class="csp-gen__policy-desc">{{ p.desc }}</span>
          <div v-if="p.enabled" class="csp-gen__sources">
            <label v-for="s in sources" :key="s.value" class="csp-gen__source">
              <input type="checkbox" :checked="p.activeSources.includes(s.value)" @change="toggleSource(p, s.value)" />
              <code>{{ s.value }}</code>
              <small>{{ s.desc }}</small>
            </label>
            <input v-model="p.custom" class="csp-gen__custom" placeholder="自定义源 (如 https://cdn.example.com)" />
          </div>
        </div>
      </div>

      <div class="csp-gen__output">
        <div class="csp-gen__output-header">
          <span>Content-Security-Policy</span>
          <button @click="copyCsp">复制</button>
        </div>
        <pre class="csp-gen__csp selectable">{{ cspHeader }}</pre>
        <p v-if="!cspHeader" class="csp-gen__hint">勾选左侧策略来生成 CSP 头</p>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { reactive, computed } from 'vue'

const sources = [
  { value: "'self'", desc: '同源' },
  { value: "'none'", desc: '禁止全部' },
  { value: "'unsafe-inline'", desc: '内联' },
  { value: "'unsafe-eval'", desc: 'eval' },
  { value: 'data:', desc: 'data URI' },
  { value: 'https:', desc: 'HTTPS' },
  { value: '*', desc: '通配符' }
]

const policies = reactive([
  { directive: 'default-src', desc: '默认策略（兜底）', enabled: true, activeSources: ["'self'"], custom: '' },
  { directive: 'script-src', desc: 'JavaScript 脚本', enabled: true, activeSources: ["'self'"], custom: '' },
  { directive: 'style-src', desc: 'CSS 样式表', enabled: true, activeSources: ["'self'", "'unsafe-inline'"], custom: '' },
  { directive: 'img-src', desc: '图片资源', enabled: true, activeSources: ["'self'", 'data:', 'https:'], custom: '' },
  { directive: 'font-src', desc: '字体文件', enabled: false, activeSources: ["'self'"], custom: '' },
  { directive: 'connect-src', desc: 'XHR/Fetch/WebSocket', enabled: false, activeSources: ["'self'"], custom: '' },
  { directive: 'media-src', desc: '音视频', enabled: false, activeSources: ["'self'"], custom: '' },
  { directive: 'frame-src', desc: 'iframe 框架', enabled: false, activeSources: ["'self'"], custom: '' },
  { directive: 'object-src', desc: 'Flash/插件', enabled: true, activeSources: ["'none'"], custom: '' },
  { directive: 'base-uri', desc: 'base 标签限制', enabled: false, activeSources: ["'self'"], custom: '' },
  { directive: 'form-action', desc: '表单提交目标', enabled: false, activeSources: ["'self'"], custom: '' }
])

function toggleSource(p, val) {
  const idx = p.activeSources.indexOf(val)
  if (idx >= 0) {
    p.activeSources.splice(idx, 1)
  } else {
    if (val === "'none'") p.activeSources = ["'none'"]
    else p.activeSources = p.activeSources.filter(s => s !== "'none'")
    p.activeSources.push(val)
  }
}

const cspHeader = computed(() => {
  const active = policies.filter(p => p.enabled)
  if (active.length === 0) return ''
  return active.map(p => {
    let srcs = [...p.activeSources]
    if (p.custom.trim()) srcs.push(p.custom.trim())
    if (srcs.length === 0) srcs = ["'self'"]
    return p.directive + ' ' + srcs.join(' ')
  }).join('; ')
})

function copyCsp() {
  navigator.clipboard?.writeText(cspHeader.value)
}
</script>

<style scoped>
.csp-gen { display: flex; gap: 16px; }
.csp-gen__policies { flex: 1; display: flex; flex-direction: column; gap: 8px; }
.csp-gen__policy { padding: 10px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.csp-gen__policy-label { display: flex; align-items: center; gap: 8px; cursor: pointer; }
.csp-gen__policy-label code { font-size: 13px; color: var(--color-primary); font-weight: 600; }
.csp-gen__policy-desc { display: block; margin-left: 24px; font-size: 11px; color: var(--text-tertiary); }
.csp-gen__sources { margin-left: 24px; margin-top: 8px; display: flex; flex-wrap: wrap; gap: 8px; align-items: center; }
.csp-gen__source { display: flex; align-items: center; gap: 4px; font-size: 12px; cursor: pointer; color: var(--text-secondary); }
.csp-gen__source code { font-size: 11px; color: var(--text-primary); }
.csp-gen__source small { font-size: 10px; color: var(--text-tertiary); }
.csp-gen__custom { padding: 4px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-primary); font-size: 12px; width: 200px; }
.csp-gen__output { width: 360px; flex-shrink: 0; display: flex; flex-direction: column; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.csp-gen__output-header { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; background: var(--bg-base); border-bottom: 1px solid var(--border-color); font-size: 12px; color: var(--text-tertiary); }
.csp-gen__output-header button { padding: 4px 10px; border: 1px solid var(--color-primary); border-radius: 4px; background: transparent; color: var(--color-primary); font-size: 11px; cursor: pointer; }
.csp-gen__csp { padding: 12px; margin: 0; font-family: monospace; font-size: 12px; color: var(--color-primary); white-space: pre-wrap; word-break: break-all; flex: 1; overflow-y: auto; }
.csp-gen__hint { padding: 20px; color: var(--text-tertiary); font-size: 13px; text-align: center; }
</style>
