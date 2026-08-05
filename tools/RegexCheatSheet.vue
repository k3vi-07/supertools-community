<template>
  <h-single-layout>
    <div class="regex-cheat">
      <input v-model="search" placeholder="搜索正则语法... (如 \\d, 锚点, 量词)" />
      <div class="regex-cheat__groups">
        <div v-for="g in filteredGroups" :key="g.title" class="regex-cheat__group">
          <h3 class="regex-cheat__title">{{ g.title }}</h3>
          <div class="regex-cheat__items">
            <div v-for="item in g.items" :key="item.syntax" class="regex-cheat__item" @click="copy(item.syntax)">
              <code class="regex-cheat__syntax selectable">{{ item.syntax }}</code>
              <span class="regex-cheat__desc">{{ item.desc }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const search = ref('')

const groups = [
  { title: '字符类', items: [
    { syntax: '.', desc: '任意单个字符（除换行）' },
    { syntax: '\\d', desc: '数字 [0-9]' },
    { syntax: '\\D', desc: '非数字' },
    { syntax: '\\w', desc: '单词字符 [a-zA-Z0-9_]' },
    { syntax: '\\W', desc: '非单词字符' },
    { syntax: '\\s', desc: '空白字符' },
    { syntax: '\\S', desc: '非空白字符' },
    { syntax: '[abc]', desc: 'a 或 b 或 c' },
    { syntax: '[^abc]', desc: '非 a/b/c 的任意字符' },
    { syntax: '[a-z]', desc: 'a 到 z 的任意字符' }
  ]},
  { title: '量词', items: [
    { syntax: '*', desc: '0 次或多次' },
    { syntax: '+', desc: '1 次或多次' },
    { syntax: '?', desc: '0 次或 1 次' },
    { syntax: '{n}', desc: '恰好 n 次' },
    { syntax: '{n,}', desc: '至少 n 次' },
    { syntax: '{n,m}', desc: 'n 到 m 次' },
    { syntax: '*?', desc: '懒惰匹配（尽可能少）' }
  ]},
  { title: '锚点与边界', items: [
    { syntax: '^', desc: '字符串开头' },
    { syntax: '$', desc: '字符串结尾' },
    { syntax: '\\b', desc: '单词边界' },
    { syntax: '\\B', desc: '非单词边界' },
    { syntax: '(?=...)', desc: '正向先行断言' },
    { syntax: '(?!...)', desc: '负向先行断言' },
    { syntax: '(?<=...)', desc: '正向后行断言' },
    { syntax: '(?<!...)', desc: '负向后行断言' }
  ]},
  { title: '分组与引用', items: [
    { syntax: '(abc)', desc: '捕获分组' },
    { syntax: '(?:abc)', desc: '非捕获分组' },
    { syntax: '(?<name>abc)', desc: '命名捕获分组' },
    { syntax: '\\1', desc: '反向引用第 1 组' },
    { syntax: '\\k<name>', desc: '反向引用命名组' },
    { syntax: 'a|b', desc: 'a 或 b（或运算）' }
  ]},
  { title: '修饰符', items: [
    { syntax: 'g', desc: '全局匹配（找所有）' },
    { syntax: 'i', desc: '不区分大小写' },
    { syntax: 'm', desc: '多行模式' },
    { syntax: 's', desc: '使 . 匹配换行符' },
    { syntax: 'u', desc: 'Unicode 模式' },
    { syntax: 'y', desc: '粘性匹配' }
  ]},
  { title: '常用转义', items: [
    { syntax: '\\n', desc: '换行符' },
    { syntax: '\\t', desc: '制表符' },
    { syntax: '\\r', desc: '回车符' },
    { syntax: '\\\\', desc: '反斜杠' },
    { syntax: '\\.', desc: '字面量点' },
    { syntax: '\\(', desc: '字面量括号' }
  ]}
]

const filteredGroups = computed(() => {
  if (!search.value.trim()) return groups
  const q = search.value.toLowerCase()
  return groups
    .map(g => ({ ...g, items: g.items.filter(i =>
      i.syntax.toLowerCase().includes(q) || i.desc.toLowerCase().includes(q) || g.title.toLowerCase().includes(q)
    )}))
    .filter(g => g.items.length > 0)
})

function copy(text) {
  navigator.clipboard?.writeText(text)
}
</script>

<style scoped>
.regex-cheat { display: flex; flex-direction: column; gap: 12px; }
.regex-cheat input { padding: 10px 14px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; outline: none; }
.regex-cheat__groups { display: flex; flex-direction: column; gap: 20px; }
.regex-cheat__title { font-size: 14px; font-weight: 600; color: var(--color-primary); margin-bottom: 8px; }
.regex-cheat__items { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 6px; }
.regex-cheat__item { display: flex; align-items: center; gap: 10px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); cursor: pointer; transition: all 0.15s; }
.regex-cheat__item:hover { border-color: var(--color-primary); }
.regex-cheat__syntax { font-family: monospace; font-size: 14px; color: var(--color-primary); font-weight: 600; min-width: 80px; }
.regex-cheat__desc { font-size: 12px; color: var(--text-secondary); }
</style>
