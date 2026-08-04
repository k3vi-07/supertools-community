<template>
  <h-single-layout>
    <div class="char-counter">
      <textarea v-model="text" class="char-counter__input selectable" placeholder="在此输入或粘贴文本..." spellcheck="false"></textarea>
      <div class="char-counter__stats">
        <div class="char-counter__stat"><span class="char-counter__num">{{ text.length }}</span><span>字符</span></div>
        <div class="char-counter__stat"><span class="char-counter__num">{{ text.replace(/\s/g,'').length }}</span><span>不含空格</span></div>
        <div class="char-counter__stat"><span class="char-counter__num">{{ words }}</span><span>单词</span></div>
        <div class="char-counter__stat"><span class="char-counter__num">{{ text ? text.split('\n').length : 0 }}</span><span>行</span></div>
        <div class="char-counter__stat"><span class="char-counter__num">{{ chinese }}</span><span>中文</span></div>
        <div class="char-counter__stat"><span class="char-counter__num">{{ bytes }}</span><span>字节</span></div>
      </div>
      <div class="char-counter__limit">
        <label>字数限制: <input type="number" v-model.number="limit" min="1" /></label>
        <div class="char-counter__bar"><div class="char-counter__fill" :style="{width: progress+'%', background: progressColor}"></div></div>
        <span>{{ text.length }} / {{ limit }}</span>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const text = ref('输入文字来统计')
const limit = ref(280)
const words = computed(() => text.value.trim() ? text.value.trim().split(/\s+/).length : 0)
const chinese = computed(() => (text.value.match(/[\u4e00-\u9fa5]/g) || []).length)
const bytes = computed(() => new Blob([text.value]).size)
const progress = computed(() => Math.min(100, (text.value.length / limit.value) * 100))
const progressColor = computed(() => progress.value > 90 ? 'var(--color-error)' : progress.value > 70 ? 'var(--color-warning)' : 'var(--color-success)')
</script>

<style scoped>
.char-counter { display: flex; flex-direction: column; gap: 12px; }
.char-counter__input { width: 100%; min-height: 120px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.char-counter__stats { display: grid; grid-template-columns: repeat(6, 1fr); gap: 8px; }
.char-counter__stat { display: flex; flex-direction: column; align-items: center; gap: 4px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); }
.char-counter__num { font-size: 22px; font-weight: 700; color: var(--color-primary); }
.char-counter__stat > span:last-child { font-size: 11px; color: var(--text-tertiary); }
.char-counter__limit { display: flex; align-items: center; gap: 12px; font-size: 13px; color: var(--text-secondary); }
.char-counter__limit input { width: 80px; padding: 4px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: var(--bg-surface); color: var(--text-primary); }
.char-counter__bar { flex: 1; height: 8px; border-radius: 4px; background: var(--bg-active); overflow: hidden; }
.char-counter__fill { height: 100%; border-radius: 4px; transition: all 0.3s; }
</style>
