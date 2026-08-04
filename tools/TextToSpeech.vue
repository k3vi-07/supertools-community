<template>
  <h-single-layout>
    <div class="tts">
      <h-multiline-input v-model="text" title="输入文本" placeholder="输入要朗读的文本..." />
      <div class="tts__controls">
        <label>语速: {{ rate }}</label>
        <input type="range" v-model.number="rate" min="0.5" max="2" step="0.1" />
        <label>音调: {{ pitch }}</label>
        <input type="range" v-model.number="pitch" min="0" max="2" step="0.1" />
        <h-button type="primary" icon="mdi:play" @click="speak">朗读</h-button>
        <h-button icon="mdi:stop" @click="stop">停止</h-button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const text = ref('Hello! 这是 SuperTools 文本转语音工具。')
const rate = ref(1)
const pitch = ref(1)

function speak(): void {
  if (!('speechSynthesis' in window)) {
    window.$he3?.message.error('浏览器不支持语音合成')
    return
  }
  window.speechSynthesis.cancel()
  const utterance = new SpeechSynthesisUtterance(text.value)
  utterance.rate = rate.value
  utterance.pitch = pitch.value
  utterance.lang = /[\u4e00-\u9fa5]/.test(text.value) ? 'zh-CN' : 'en-US'
  window.speechSynthesis.speak(utterance)
}

function stop(): void {
  window.speechSynthesis.cancel()
}
</script>

<style scoped>
.tts { display: flex; flex-direction: column; gap: 16px; }
.tts__controls { display: flex; align-items: center; gap: 12px; flex-wrap: wrap; }
.tts__controls label { font-size: 12px; color: var(--text-secondary); }
.tts__controls input[type=range] { accent-color: var(--color-primary); width: 100px; }
</style>
