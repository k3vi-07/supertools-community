<template>
  <h-single-layout>
    <div class="regex-test">
      <!-- 正则输入 -->
      <div class="regex-test__pattern">
        <span class="regex-test__slash">/</span>
        <input v-model="pattern" class="regex-test__pattern-input" placeholder="输入正则表达式..." spellcheck="false" />
        <span class="regex-test__slash">/</span>
        <input v-model="flags" class="regex-test__flags" placeholder="gim" spellcheck="false" />
        <button v-for="f in flagOptions" :key="f" class="regex-test__flag-btn" :class="{ active: flags.includes(f) }" @click="toggleFlag(f)">{{ f }}</button>
      </div>

      <div v-if="regexError" class="regex-test__error">⚠️ {{ regexError }}</div>

      <!-- 匹配结果统计 -->
      <div v-if="!regexError && matches" class="regex-test__stats">
        <span>匹配数: <strong>{{ matches.length }}</strong></span>
        <span v-if="groups.length > 0">捕获组: <strong>{{ groups.length }}</strong></span>
      </div>

      <!-- 文本输入 + 高亮 -->
      <div class="regex-test__io">
        <div class="regex-test__panel">
          <div class="regex-test__panel-header"><span>测试文本</span></div>
          <textarea v-model="testText" class="regex-test__text selectable" spellcheck="false"></textarea>
        </div>
        <div class="regex-test__panel">
          <div class="regex-test__panel-header"><span>匹配结果</span></div>
          <div class="regex-test__highlight selectable" v-html="highlightedText"></div>
        </div>
      </div>

      <!-- 匹配详情 -->
      <div v-if="!regexError && matches.length > 0" class="regex-test__details">
        <div v-for="(m, i) in matches.slice(0, 50)" :key="i" class="regex-test__match-item">
          <span class="regex-test__match-index">#{{ i + 1 }}</span>
          <code class="regex-test__match-text">{{ m.text }}</code>
          <span class="regex-test__match-pos">位置 {{ m.index }}-{{ m.index + m.text.length }}</span>
          <span v-if="m.groups && m.groups.length > 0" class="regex-test__match-groups">
            <span v-for="(g, gi) in m.groups" :key="gi" class="regex-test__group">[{{ gi + 1 }}] {{ g || '∅' }}</span>
          </span>
        </div>
        <div v-if="matches.length > 50" class="regex-test__more">... 还有 {{ matches.length - 50 }} 个匹配</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const pattern = ref('\\d+\\.?\\d*')
const flags = ref('g')
const testText = ref('订单号: 20240115, 金额: 99.50 元, 数量: 3 个, 折扣: 0.85\n联系方式: 13800138000, 备用: 400-123-4567')

const flagOptions = ['g', 'i', 'm', 's', 'u']

function toggleFlag(f) {
  if (flags.value.includes(f)) {
    flags.value = flags.value.replace(f, '')
  } else {
    flags.value += f
  }
}

const compiledRegex = computed(() => {
  if (!pattern.value) return null
  try {
    return new RegExp(pattern.value, flags.value)
  } catch (e) {
    return null
  }
})

const regexError = computed(() => {
  if (!pattern.value) return ''
  try {
    new RegExp(pattern.value, flags.value)
    return ''
  } catch (e) {
    return e.message
  }
})

const matches = computed(() => {
  if (!compiledRegex.value || !testText.value) return []
  const regex = new RegExp(pattern.value, flags.value.includes('g') ? flags.value : flags.value + 'g')
  const results = []
  let m
  while ((m = regex.exec(testText.value)) !== null) {
    results.push({
      text: m[0],
      index: m.index,
      groups: m.slice(1)
    })
    if (m[0] === '') regex.lastIndex++
  }
  return results
})

const groups = computed(() => {
  if (matches.value.length === 0) return []
  return matches.value[0].groups
})

function escapeHtml(s) {
  return s.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
}

const highlightedText = computed(() => {
  if (!compiledRegex.value || !testText.value || regexError.value) {
    return escapeHtml(testText.value)
  }
  if (matches.value.length === 0) return escapeHtml(testText.value)
  let result = ''
  let lastIndex = 0
  for (const m of matches.value) {
    result += escapeHtml(testText.value.substring(lastIndex, m.index))
    result += '<mark>' + escapeHtml(m.text) + '</mark>'
    lastIndex = m.index + m.text.length
  }
  result += escapeHtml(testText.value.substring(lastIndex))
  return result
})
</script>

<style scoped>
.regex-test { display: flex; flex-direction: column; gap: 12px; }
.regex-test__pattern { display: flex; align-items: center; gap: 4px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.regex-test__slash { font-size: 20px; color: var(--text-tertiary); font-family: monospace; }
.regex-test__pattern-input { flex: 1; border: none; background: transparent; color: var(--color-primary); font-family: monospace; font-size: 16px; outline: none; }
.regex-test__flags { width: 50px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-primary); font-family: monospace; font-size: 13px; padding: 4px 6px; text-align: center; outline: none; }
.regex-test__flag-btn { padding: 4px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-base); color: var(--text-tertiary); font-family: monospace; font-size: 12px; cursor: pointer; }
.regex-test__flag-btn.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.regex-test__error { padding: 10px 12px; border-radius: 8px; background: rgba(239,68,68,0.1); color: #ef4444; font-size: 13px; }
.regex-test__stats { display: flex; gap: 20px; font-size: 13px; color: var(--text-secondary); }
.regex-test__stats strong { color: var(--color-primary); }
.regex-test__io { display: flex; gap: 12px; }
.regex-test__panel { flex: 1; display: flex; flex-direction: column; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.regex-test__panel-header { padding: 6px 10px; background: var(--bg-base); border-bottom: 1px solid var(--border-color); font-size: 11px; color: var(--text-tertiary); }
.regex-test__text { height: 160px; padding: 10px; border: none; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.regex-test__highlight { height: 160px; padding: 10px; overflow-y: auto; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; white-space: pre-wrap; word-break: break-all; }
.regex-test__highlight :deep(mark) { background: rgba(245,158,11,0.3); color: #f59e0b; border-radius: 2px; padding: 1px 2px; }
.regex-test__details { display: flex; flex-direction: column; gap: 4px; max-height: 200px; overflow-y: auto; }
.regex-test__match-item { display: flex; align-items: center; gap: 8px; padding: 6px 10px; border-radius: 6px; background: var(--bg-surface); font-size: 12px; }
.regex-test__match-index { color: var(--text-tertiary); font-size: 11px; min-width: 30px; }
.regex-test__match-text { font-family: monospace; color: var(--color-primary); }
.regex-test__match-pos { color: var(--text-tertiary); font-size: 11px; }
.regex-test__match-groups { display: flex; gap: 6px; }
.regex-test__group { padding: 1px 6px; border-radius: 3px; background: var(--bg-base); font-size: 11px; color: var(--text-secondary); }
.regex-test__more { padding: 8px; text-align: center; color: var(--text-tertiary); font-size: 12px; }
</style>
