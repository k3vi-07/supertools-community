<template>
  <h-single-layout>
    <div class="float-conv">
      <div class="float-conv__input"><label>十进制浮点数</label><input v-model="dec" type="number" step="any" @input="calcFromDec" /></div>
      <div class="float-conv__divider">⇅</div>
      <div class="float-conv__input"><label>IEEE 754 十六进制</label><input v-model="hex" @input="calcFromHex" /></div>
      <div class="float-conv__bits">
        <div class="float-conv__bit-section">
          <span class="float-conv__label">符号位</span>
          <span class="float-conv__bit" :class="{on: bits.sign}">{{ bits.sign }}</span>
        </div>
        <div class="float-conv__bit-section">
          <span class="float-conv__label">指数 (8位)</span>
          <span class="float-conv__bits-exp">{{ bits.exponent }}</span>
        </div>
        <div class="float-conv__bit-section">
          <span class="float-conv__label">尾数 (23位)</span>
          <span class="float-conv__bits-man">{{ bits.mantissa }}</span>
        </div>
      </div>
      <div class="float-conv__info">
        <div class="float-conv__row"><span>二进制表示</span><code class="selectable">{{ bits.full }}</code></div>
        <div class="float-conv__row"><span>十六进制</span><code class="selectable">{{ hex }}</code></div>
        <div class="float-conv__row"><span>十进制值</span><code class="selectable">{{ dec }}</code></div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'
const dec = ref('3.14')
const hex = ref('4048F5C3')

const bits = computed(() => {
  const buf = new ArrayBuffer(4)
  const f32 = new Float32Array(buf)
  const u32 = new Uint32Array(buf)
  f32[0] = parseFloat(dec.value) || 0
  const val = u32[0]
  const full = val.toString(2).padStart(32, '0')
  return {
    sign: (val >>> 31) & 1,
    exponent: full.substring(1, 9),
    mantissa: full.substring(9),
    full
  }
})

function calcFromDec() {
  const buf = new ArrayBuffer(4)
  new Float32Array(buf)[0] = parseFloat(dec.value) || 0
  hex.value = '0x' + new Uint32Array(buf)[0].toString(16).toUpperCase().padStart(8, '0')
}

function calcFromHex() {
  const h = hex.value.replace('0x', '')
  const buf = new ArrayBuffer(4)
  new Uint32Array(buf)[0] = parseInt(h, 16)
  dec.value = String(new Float32Array(buf)[0])
}

calcFromDec()
</script>

<style scoped>
.float-conv { display: flex; flex-direction: column; gap: 12px; }
.float-conv__input { display: flex; flex-direction: column; gap: 4px; }
.float-conv__input label { font-size: 12px; color: var(--text-secondary); }
.float-conv__input input { padding: 8px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 14px; }
.float-conv__divider { text-align: center; font-size: 20px; color: var(--text-tertiary); }
.float-conv__bits { display: flex; gap: 8px; padding: 16px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.float-conv__bit-section { display: flex; flex-direction: column; gap: 4px; }
.float-conv__label { font-size: 11px; color: var(--text-tertiary); }
.float-conv__bit { font-family: monospace; font-size: 14px; padding: 2px 6px; border-radius: 4px; background: var(--bg-active); color: var(--text-secondary); }
.float-conv__bit.on { color: var(--color-error); }
.float-conv__bits-exp, .float-conv__bits-man { font-family: monospace; font-size: 13px; color: var(--color-primary); word-break: break-all; }
.float-conv__info { display: flex; flex-direction: column; gap: 6px; }
.float-conv__row { display: flex; justify-content: space-between; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; }
.float-conv__row span { color: var(--text-tertiary); }
.float-conv__row code { font-family: monospace; color: var(--color-primary); }
</style>
