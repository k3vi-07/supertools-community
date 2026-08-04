<template>
  <h-single-layout>
    <div class="key-codes">
      <div class="key-codes__input">按下任意键查看 keyCode...</div>
      <div v-if="lastKey" class="key-codes__result">
        <div class="key-codes__display">
          <div class="key-codes__key">{{ lastKey.key }}</div>
        </div>
        <div class="key-codes__info">
          <div class="key-codes__row"><span>key</span><code>{{ lastKey.key }}</code></div>
          <div class="key-codes__row"><span>code</span><code>{{ lastKey.code }}</code></div>
          <div class="key-codes__row"><span>keyCode</span><code class="key-codes__highlight">{{ lastKey.keyCode }}</code></div>
          <div class="key-codes__row"><span>which</span><code>{{ lastKey.keyCode }}</code></div>
          <div class="key-codes__row"><span>altKey</span><code>{{ lastKey.altKey }}</code></div>
          <div class="key-codes__row"><span>ctrlKey</span><code>{{ lastKey.ctrlKey }}</code></div>
          <div class="key-codes__row"><span>shiftKey</span><code>{{ lastKey.shiftKey }}</code></div>
          <div class="key-codes__row"><span>metaKey</span><code>{{ lastKey.metaKey }}</code></div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
const lastKey = ref<KeyboardEvent | null>(null)

function handler(e: KeyboardEvent): void {
  e.preventDefault()
  lastKey.value = e
}

onMounted(() => window.addEventListener('keydown', handler))
onUnmounted(() => window.removeEventListener('keydown', handler))
</script>

<style scoped>
.key-codes { display: flex; flex-direction: column; gap: 16px; }
.key-codes__input { padding: 20px; text-align: center; border: 2px dashed var(--border-color); border-radius: 8px; color: var(--text-tertiary); font-size: 14px; }
.key-codes__display { display: flex; justify-content: center; padding: 20px; }
.key-codes__key { min-width: 120px; padding: 20px; text-align: center; background: var(--color-primary); color: white; border-radius: 8px; font-size: 28px; font-weight: 700; font-family: monospace; box-shadow: 0 4px 12px rgba(124,58,237,0.3); }
.key-codes__info { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; }
.key-codes__row { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); font-size: 13px; }
.key-codes__row span { color: var(--text-tertiary); }
.key-codes__row code { font-family: monospace; color: var(--text-primary); }
.key-codes__highlight { color: var(--color-primary) !important; font-weight: 700; }
</style>
