<template>
  <h-single-layout>
    <div class="n2c">
      <h-multiline-input v-model="input" title="输入数字" placeholder="输入数字，如 12345.67" />
      <div class="n2c__result">
        <div class="n2c__result-header">
          <span>中文大写金额</span>
          <button v-if="output" class="n2c__copy" @click="copy(output)">
            <h-icon icon="mdi:content-copy" :size="14" /> 复制
          </button>
        </div>
        <div class="n2c__output selectable">{{ output || '—' }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const input = ref('12345.67')

const digits = ['零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖']
const units = ['', '拾', '佰', '仟']
const bigUnits = ['', '万', '亿', '兆']

const output = computed(() => {
  const num = parseFloat(input.value.trim())
  if (isNaN(num)) return '请输入有效数字'
  if (num === 0) return '零元整'

  const isNegative = num < 0
  const absNum = Math.abs(num)
  const [intPart, decPart] = absNum.toFixed(2).split('.')

  let result = convertInteger(intPart)
  if (decPart && decPart !== '00') {
    const jiao = parseInt(decPart[0])
    const fen = parseInt(decPart[1])
    if (jiao > 0) result += digits[jiao] + '角'
    if (fen > 0) result += digits[fen] + '分'
  } else {
    result += '整'
  }

  return (isNegative ? '负' : '') + result
})

function convertInteger(intStr: string): string {
  let result = ''
  const len = intStr.length
  const bigUnitIndex = Math.ceil(len / 4) - 1

  for (let i = 0; i < len; i++) {
    const digit = parseInt(intStr[i])
    const pos = len - i - 1
    const unitPos = pos % 4
    const currentBigUnit = Math.floor(pos / 4)

    if (digit === 0) {
      if (unitPos === 0 && currentBigUnit < bigUnitIndex) {
        result += bigUnits[bigUnitIndex - currentBigUnit]
      }
      if (result && !result.endsWith('零') && !result.endsWith('万') && !result.endsWith('亿')) {
        result += '零'
      }
    } else {
      result += digits[digit] + units[unitPos]
      if (unitPos === 0) {
        result += bigUnits[bigUnitIndex - currentBigUnit]
      }
    }
  }

  result = result.replace(/零+/g, '零').replace(/零(万|亿)/g, '$1').replace(/^零/, '').replace(/零$/, '')
  return result + '元'
}

function copy(text: string): void {
  window.$he3?.copyText(text)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.n2c { display: flex; flex-direction: column; gap: 16px; }
.n2c__result { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.n2c__result-header { display: flex; align-items: center; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; font-weight: 600; color: var(--text-secondary); }
.n2c__copy { display: flex; align-items: center; gap: 4px; padding: 2px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; }
.n2c__copy:hover { background: var(--bg-hover); }
.n2c__output { padding: 16px; font-size: 18px; font-weight: 600; color: var(--color-primary); }
</style>
