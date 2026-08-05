<template>
  <h-single-layout>
    <div class="symbols">
      <h-input v-model="search" placeholder="搜索符号... (如 arrow, star, check)" />
      <div class="symbols__grid">
        <button v-for="sym in filtered" :key="sym.char" class="symbols__item" :title="sym.name" @click="copy(sym)">
          <span class="symbols__char">{{ sym.char }}</span>
          <span class="symbols__name">{{ sym.name }}</span>
        </button>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const search = ref('')

const symbols= [
  {char:'✓',name:'check'},{char:'✗',name:'cross'},{char:'★',name:'star'},{char:'☆',name:'star empty'},
  {char:'♥',name:'heart'},{char:'♦',name:'diamond'},{char:'♣',name:'club'},{char:'♠',name:'spade'},
  {char:'☀',name:'sun'},{char:'☁',name:'cloud'},{char:'☂',name:'umbrella'},{char:'☃',name:'snowman'},
  {char:'☎',name:'phone'},{char:'✉',name:'email'},{char:'✎',name:'pencil'},{char:'✂',name:'scissors'},
  {char:'☛',name:'pointer'},{char:'☝',name:'point up'},{char:'☟',name:'point down'},{char:'☜',name:'point left'},
  {char:'☞',name:'point right'},{char:'✪',name:'star circle'},{char:'✰',name:'star sparkle'},
  {char:'⚛',name:'atom'},{char:'⚡',name:'lightning'},{char:'♾',name:'infinity'},{char:'√',name:'sqrt'},
  {char:'∑',name:'sum'},{char:'∏',name:'product'},{char:'∂',name:'partial'},{char:'∫',name:'integral'},
  {char:'≈',name:'approx'},{char:'≠',name:'not equal'},{char:'≤',name:'less equal'},{char:'≥',name:'greater equal'},
  {char:'∈',name:'element'},{char:'∉',name:'not element'},{char:'⊂',name:'subset'},{char:'⊃',name:'superset'},
  {char:'∪',name:'union'},{char:'∩',name:'intersection'},{char:'∅',name:'empty set'},{char:'π',name:'pi'},
  {char:'→',name:'arrow right'},{char:'←',name:'arrow left'},{char:'↑',name:'arrow up'},{char:'↓',name:'arrow down'},
  {char:'↔',name:'arrow h'},{char:'↕',name:'arrow v'},{char:'⇒',name:'implies'},{char:'⇐',name:'implied by'},
  {char:'⇔',name:'iff'},{char:'°',name:'degree'},{char:'±',name:'plus minus'},{char:'×',name:'multiply'},
  {char:'÷',name:'divide'},{char:'µ',name:'micro'},{char:'Ω',name:'ohm'},{char:'♪',name:'note'},
  {char:'♫',name:'notes'},{char:'☀',name:'sun'},{char:'☽',name:'moon'},{char:'⚙',name:'gear'},
  {char:'⚠',name:'warning'},{char:'⚡',name:'zap'},{char:'♔',name:'white king'},{char:'♕',name:'white queen'},
  {char:'♖',name:'white rook'},{char:'♗',name:'white bishop'},{char:'♘',name:'white knight'},{char:'♙',name:'white pawn'},
  {char:'①',name:'circled 1'},{char:'②',name:'circled 2'},{char:'③',name:'circled 3'},{char:'④',name:'circled 4'},
  {char:'⑤',name:'circled 5'},{char:'⑥',name:'circled 6'},{char:'⑦',name:'circled 7'},{char:'⑧',name:'circled 8'},
  {char:'⑨',name:'circled 9'},{char:'⑩',name:'circled 10'},{char:'½',name:'one half'},{char:'⅓',name:'one third'},
  {char:'¼',name:'one quarter'},{char:'¾',name:'three quarters'},{char:'‰',name:'per mille'},{char:'§',name:'section'},
  {char:'¶',name:'pilcrow'},{char:'†',name:'dagger'},{char:'‡',name:'double dagger'},{char:'•',name:'bullet'},
  {char:'…',name:'ellipsis'},{char:'—',name:'em dash'},{char:'–',name:'en dash'}
]
const filtered = computed(() => {
  if (!search.value.trim()) return symbols
  const q = search.value.toLowerCase()
  return symbols.filter((s) => s.name.includes(q) || s.char.includes(q))
})
function copy(sym) {
  window.$he3?.copyText(sym.char)
  window.$he3?.message.success(`已复制 ${sym.char}`)
}
</script>

<style scoped>
.symbols { display: flex; flex-direction: column; gap: 12px; }
.symbols input { width: 100%; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.symbols__grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(80px, 1fr)); gap: 6px; max-height: 500px; overflow-y: auto; }
.symbols__item { display: flex; flex-direction: column; align-items: center; gap: 2px; padding: 8px 4px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.symbols__item:hover { border-color: var(--color-primary); transform: translateY(-2px); }
.symbols__char { font-size: 20px; }
.symbols__name { font-size: 9px; color: var(--text-tertiary); }
</style>
