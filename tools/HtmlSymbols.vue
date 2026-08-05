<template>
  <h-single-layout>
    <div class="html-symbols">
      <input v-model="search" placeholder="搜索符号名称... (如 copy, trade, arrow)" />
      <div class="html-symbols__grid">
        <div v-for="sym in filtered" :key="sym.name" class="html-symbols__item" @click="copy(sym)">
          <span class="html-symbols__char">{{ sym.char }}</span>
          <code class="html-symbols__code">{{ sym.entity }}</code>
          <span class="html-symbols__name">{{ sym.name }}</span>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const search = ref('')
const symbols: Sym[] = [
  {char:'©',entity:'&copy;',name:'copyright 版权'},{char:'®',entity:'&reg;',name:'registered 注册商标'},
  {char:'™',entity:'&trade;',name:'trademark 商标'},{char:'€',entity:'&euro;',name:'euro 欧元'},
  {char:'£',entity:'&pound;',name:'pound 英镑'},{char:'¥',entity:'&yen;',name:'yen 人民币'},
  {char:'¢',entity:'&cent;',name:'cent 美分'},{char:'§',entity:'&sect;',name:'section 章节'},
  {char:'¶',entity:'&para;',name:'paragraph 段落'},{char:'•',entity:'&bull;',name:'bullet 项目符号'},
  {char:'…',entity:'&hellip;',name:'ellipsis 省略号'},{char:'′',entity:'&prime;',name:'prime 撇'},
  {char:'″',entity:'&Prime;',name:'double prime 双撇'},{char:'°',entity:'&deg;',name:'degree 度'},
  {char:'±',entity:'&plusmn;',name:'plus minus 正负'},{char:'×',entity:'&times;',name:'multiply 乘'},
  {char:'÷',entity:'&divide;',name:'divide 除'},{char:'√',entity:'&radic;',name:'square root 根号'},
  {char:'∞',entity:'&infin;',name:'infinity 无穷'},{char:'≈',entity:'&asymp;',name:'approx 约等于'},
  {char:'≠',entity:'&ne;',name:'not equal 不等'},{char:'≤',entity:'&le;',name:'less equal 小于等于'},
  {char:'≥',entity:'&ge;',name:'greater equal 大于等于'},{char:'←',entity:'&larr;',name:'left arrow 左箭头'},
  {char:'↑',entity:'&uarr;',name:'up arrow 上箭头'},{char:'→',entity:'&rarr;',name:'right arrow 右箭头'},
  {char:'↓',entity:'&darr;',name:'down arrow 下箭头'},{char:'↔',entity:'&harr;',name:'left right arrow'},
  {char:'⇐',entity:'&lArr;',name:'left double arrow'},{char:'⇒',entity:'&rArr;',name:'right double arrow'},
  {char:'⇔',entity:'&hArr;',name:'double arrow'},{char:'♠',entity:'&spades;',name:'spade 黑桃'},
  {char:'♣',entity:'&clubs;',name:'club 梅花'},{char:'♥',entity:'&hearts;',name:'heart 红心'},
  {char:'♦',entity:'&diams;',name:'diamond 方块'},{char:'α',entity:'&alpha;',name:'alpha'},
  {char:'β',entity:'&beta;',name:'beta'},{char:'γ',entity:'&gamma;',name:'gamma'},
  {char:'δ',entity:'&delta;',name:'delta'},{char:'ε',entity:'&epsilon;',name:'epsilon'},
  {char:'π',entity:'&pi;',name:'pi 圆周率'},{char:'Σ',entity:'&Sigma;',name:'Sigma 求和'},
  {char:'λ',entity:'&lambda;',name:'lambda'},{char:'μ',entity:'&mu;',name:'mu 微'},
  {char:'Ω',entity:'&Omega;',name:'Omega 欧姆'},{char:'&',entity:'&amp;',name:'ampersand 和'},
  {char:'<',entity:'&lt;',name:'less than 小于'},{char:'>',entity:'&gt;',name:'greater than 大于'},
  {char:'"',entity:'&quot;',name:'quotation 引号'},{char:''',entity:'&apos;',name:'apostrophe 撇号'},
  {char:' ',entity:'&nbsp;',name:'non-breaking space 不换行空格'},{char:'—',entity:'&mdash;',name:'em dash 破折号'},
  {char:'–',entity:'&ndash;',name:'en dash 短破折号'},{char:'«',entity:'&laquo;',name:'left angle quote'},
  {char:'»',entity:'&raquo;',name:'right angle quote'},{char:'‰',entity:'&permil;',name:'per mille 千分号'},
  {char:'†',entity:'&dagger;',name:'dagger 匕首'},{char:'‡',entity:'&Dagger;',name:'double dagger'},
]
const filtered = computed(() => {
  if (!search.value.trim()) return symbols
  const q = search.value.toLowerCase()
  return symbols.filter((s) => s.name.includes(q) || s.entity.includes(q) || s.char.includes(q))
})
function copy(sym: Sym){
  window.$he3?.copyText(sym.entity)
  window.$he3?.message.success(`已复制 ${sym.entity} (${sym.char})`)
}
</script>

<style scoped>
.html-symbols { display: flex; flex-direction: column; gap: 12px; }
.html-symbols input { padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; }
.html-symbols__grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(120px, 1fr)); gap: 8px; max-height: 500px; overflow-y: auto; }
.html-symbols__item { display: flex; flex-direction: column; align-items: center; gap: 4px; padding: 10px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.html-symbols__item:hover { border-color: var(--color-primary); transform: translateY(-2px); }
.html-symbols__char { font-size: 24px; }
.html-symbols__code { font-family: monospace; font-size: 11px; color: var(--color-primary); }
.html-symbols__name { font-size: 10px; color: var(--text-tertiary); text-align: center; }
</style>
