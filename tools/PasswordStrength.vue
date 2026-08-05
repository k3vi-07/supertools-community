<template>
  <h-single-layout>
    <div class="pw-strength">
      <div class="pw-strength__input-area">
        <input v-model="password" :type="showPassword ? 'text' : 'password'" class="pw-strength__input" placeholder="输入密码测试强度..." />
        <button class="pw-strength__toggle" @click="showPassword = !showPassword">{{ showPassword ? '隐藏' : '显示' }}</button>
      </div>

      <div class="pw-strength__meter">
        <div class="pw-strength__bar-bg">
          <div class="pw-strength__bar" :style="{ width: score + '%', background: scoreColor }"></div>
        </div>
        <span class="pw-strength__label" :style="{ color: scoreColor }">{{ scoreLabel }}</span>
      </div>

      <div class="pw-strength__stats">
        <div class="pw-strength__stat">
          <span class="pw-strength__stat-label">长度</span>
          <span class="pw-strength__stat-value">{{ password.length }}</span>
        </div>
        <div class="pw-strength__stat">
          <span class="pw-strength__stat-label">熵值</span>
          <span class="pw-strength__stat-value">{{ entropy.toFixed(1) }} bits</span>
        </div>
        <div class="pw-strength__stat">
          <span class="pw-strength__stat-label">破解时间</span>
          <span class="pw-strength__stat-value">{{ crackTime }}</span>
        </div>
      </div>

      <div class="pw-strength__checks">
        <div class="pw-strength__check" :class="{ pass: hasLower }">
          <span>{{ hasLower ? '✅' : '⬜' }}</span> 小写字母 a-z
        </div>
        <div class="pw-strength__check" :class="{ pass: hasUpper }">
          <span>{{ hasUpper ? '✅' : '⬜' }}</span> 大写字母 A-Z
        </div>
        <div class="pw-strength__check" :class="{ pass: hasNumber }">
          <span>{{ hasNumber ? '✅' : '⬜' }}</span> 数字 0-9
        </div>
        <div class="pw-strength__check" :class="{ pass: hasSymbol }">
          <span>{{ hasSymbol ? '✅' : '⬜' }}</span> 特殊符号
        </div>
        <div class="pw-strength__check" :class="{ pass: password.length >= 12 }">
          <span>{{ password.length >= 12 ? '✅' : '⬜' }}</span> 至少 12 位
        </div>
      </div>

      <div v-if="issues.length > 0" class="pw-strength__issues">
        <div v-for="issue in issues" :key="issue" class="pw-strength__issue">⚠️ {{ issue }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const password = ref('MyP@ssw0rd2024')
const showPassword = ref(true)

const hasLower = computed(() => /[a-z]/.test(password.value))
const hasUpper = computed(() => /[A-Z]/.test(password.value))
const hasNumber = computed(() => /\d/.test(password.value))
const hasSymbol = computed(() => /[^a-zA-Z0-9]/.test(password.value))

const charsetSize = computed(() => {
  let size = 0
  if (hasLower.value) size += 26
  if (hasUpper.value) size += 26
  if (hasNumber.value) size += 10
  if (hasSymbol.value) size += 32
  return size || 1
})

const entropy = computed(() => {
  if (!password.value) return 0
  return password.value.length * Math.log2(charsetSize.value)
})

const score = computed(() => {
  if (!password.value) return 0
  return Math.min(100, Math.round((entropy.value / 100) * 100))
})

const scoreColor = computed(() => {
  if (score.value < 30) return '#ef4444'
  if (score.value < 55) return '#f59e0b'
  if (score.value < 75) return '#3b82f6'
  return '#22c55e'
})

const scoreLabel = computed(() => {
  if (!password.value) return '未输入'
  if (score.value < 30) return '弱'
  if (score.value < 55) return '一般'
  if (score.value < 75) return '强'
  return '非常强'
})

const crackTime = computed(() => {
  if (!password.value) return '-'
  // 假设 10 亿次/秒（现代 GPU）
  const guesses = Math.pow(2, entropy.value)
  const seconds = guesses / 1e9
  if (seconds < 1) return '即时'
  if (seconds < 60) return Math.round(seconds) + ' 秒'
  if (seconds < 3600) return Math.round(seconds / 60) + ' 分钟'
  if (seconds < 86400) return Math.round(seconds / 3600) + ' 小时'
  if (seconds < 31536000) return Math.round(seconds / 86400) + ' 天'
  const years = seconds / 31536000
  if (years < 1000) return Math.round(years) + ' 年'
  if (years < 1e6) return Math.round(years / 1000) + ' 千年'
  return '数百万年+'
})

const issues = computed(() => {
  const list = []
  if (password.value.length > 0 && password.value.length < 8) list.push('密码太短，建议至少 8 位')
  if (/(.)\1{2,}/.test(password.value)) list.push('包含连续重复字符（如 aaa）')
  if (/^(123|abc|qwe|password|admin|111|000)/i.test(password.value)) list.push('包含常见弱密码模式')
  if (/^[0-9]+$/.test(password.value)) list.push('纯数字密码极易被破解')
  if (!hasSymbol.value && password.value.length > 0) list.push('建议添加特殊符号增强强度')
  return list
})
</script>

<style scoped>
.pw-strength { display: flex; flex-direction: column; gap: 16px; }
.pw-strength__input-area { display: flex; gap: 8px; }
.pw-strength__input { flex: 1; padding: 12px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 16px; font-family: monospace; outline: none; }
.pw-strength__toggle { padding: 8px 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-secondary); font-size: 13px; cursor: pointer; }
.pw-strength__meter { display: flex; align-items: center; gap: 12px; }
.pw-strength__bar-bg { flex: 1; height: 10px; border-radius: 5px; background: var(--bg-base); overflow: hidden; }
.pw-strength__bar { height: 100%; border-radius: 5px; transition: width 0.3s, background 0.3s; }
.pw-strength__label { font-size: 16px; font-weight: 700; min-width: 60px; text-align: right; }
.pw-strength__stats { display: flex; gap: 12px; }
.pw-strength__stat { flex: 1; display: flex; flex-direction: column; gap: 4px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.pw-strength__stat-label { font-size: 11px; color: var(--text-tertiary); }
.pw-strength__stat-value { font-size: 18px; font-weight: 700; color: var(--text-primary); font-family: monospace; }
.pw-strength__checks { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 8px; }
.pw-strength__check { display: flex; align-items: center; gap: 6px; padding: 8px 12px; border-radius: 6px; background: var(--bg-surface); font-size: 13px; color: var(--text-secondary); }
.pw-strength__check.pass { color: var(--color-success, #22c55e); }
.pw-strength__issues { display: flex; flex-direction: column; gap: 6px; }
.pw-strength__issue { padding: 8px 12px; border-radius: 6px; background: rgba(245,158,11,0.1); font-size: 13px; color: #f59e0b; }
</style>
