<template>
  <h-single-layout>
    <div class="html2md">
      <div class="html2md__io">
        <div class="html2md__panel">
          <div class="html2md__header"><span>HTML 输入</span></div>
          <textarea v-model="input" class="html2md__ta selectable" spellcheck="false"></textarea>
        </div>
        <div class="html2md__panel">
          <div class="html2md__header"><span>Markdown 输出</span><button @click="copyMd">复制</button></div>
          <div class="html2md__output selectable">{{ output }}</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('<h1>Hello World</h1>\n<p>This is <strong>bold</strong> and <em>italic</em> text.</p>\n<h2>List</h2>\n<ul>\n<li>Item 1</li>\n<li>Item 2</li>\n</ul>\n<h2>Code</h2>\n<pre><code>const x = 42;</code></pre>\n<h2>Link</h2>\n<p>Visit <a href="https://github.com">GitHub</a></p>\n<blockquote>This is a quote.</blockquote>')

const output = computed(() => {
  let md = input.value
  // pre/code 块
  md = md.replace(/<pre[^>]*><code[^>]*>([\s\S]*?)<\/code><\/pre>/gi, (m, code) => {
    return '```\n' + decodeEntities(code.trim()) + '\n```'
  })
  // 行内 code
  md = md.replace(/<code[^>]*>([\s\S]*?)<\/code>/gi, (m, code) => '`' + decodeEntities(code) + '`')
  // 标题
  md = md.replace(/<h1[^>]*>([\s\S]*?)<\/h1>/gi, '# $1\n')
  md = md.replace(/<h2[^>]*>([\s\S]*?)<\/h2>/gi, '## $1\n')
  md = md.replace(/<h3[^>]*>([\s\S]*?)<\/h3>/gi, '### $1\n')
  md = md.replace(/<h4[^>]*>([\s\S]*?)<\/h4>/gi, '#### $1\n')
  md = md.replace(/<h5[^>]*>([\s\S]*?)<\/h5>/gi, '##### $1\n')
  md = md.replace(/<h6[^>]*>([\s\S]*?)<\/h6>/gi, '###### $1\n')
  // 粗体/斜体
  md = md.replace(/<(strong|b)[^>]*>([\s\S]*?)<\/\1>/gi, '**$2**')
  md = md.replace(/<(em|i)[^>]*>([\s\S]*?)<\/\1>/gi, '*$2*')
  // 链接
  md = md.replace(/<a[^>]*href="([^"]*)"[^>]*>([\s\S]*?)<\/a>/gi, '[$2]($1)')
  // 图片
  md = md.replace(/<img[^>]*src="([^"]*)"[^>]*alt="([^"]*)"[^>]*\/?>/gi, '![$2]($1)')
  md = md.replace(/<img[^>]*src="([^"]*)"[^>]*\/?>/gi, '![]($1)')
  // 引用
  md = md.replace(/<blockquote[^>]*>([\s\S]*?)<\/blockquote>/gi, (m, c) => {
    return c.trim().split('\n').map(l => '> ' + l).join('\n') + '\n'
  })
  // 无序列表
  md = md.replace(/<ul[^>]*>([\s\S]*?)<\/ul>/gi, (m, c) => {
    return c.replace(/<li[^>]*>([\s\S]*?)<\/li>/gi, '- $1\n').trim() + '\n'
  })
  // 有序列表
  md = md.replace(/<ol[^>]*>([\s\S]*?)<\/ol>/gi, (m, c) => {
    let i = 1
    return c.replace(/<li[^>]*>([\s\S]*?)<\/li>/gi, () => (i++) + '. $1\n').trim() + '\n'
  })
  // 水平线
  md = md.replace(/<hr[^>]*\/?>/gi, '---\n')
  // 段落和换行
  md = md.replace(/<p[^>]*>([\s\S]*?)<\/p>/gi, '$1\n\n')
  md = md.replace(/<br\s*\/?>/gi, '\n')
  // 残留 div
  md = md.replace(/<\/?div[^>]*>/gi, '')
  // 清理多余空行
  md = md.replace(/\n{3,}/g, '\n\n').trim()
  return md
})

function decodeEntities(s) {
  return s.replace(/&amp;/g, '&').replace(/&lt;/g, '<').replace(/&gt;/g, '>').replace(/&quot;/g, '"').replace(/&#39;/g, "'")
}

function copyMd() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.html2md { display: flex; flex-direction: column; gap: 12px; }
.html2md__io { display: flex; gap: 12px; }
.html2md__panel { flex: 1; display: flex; flex-direction: column; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.html2md__header { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; background: var(--bg-base); border-bottom: 1px solid var(--border-color); font-size: 12px; color: var(--text-tertiary); }
.html2md__header button { padding: 4px 10px; border: 1px solid var(--color-primary); border-radius: 4px; background: transparent; color: var(--color-primary); font-size: 11px; cursor: pointer; }
.html2md__ta { height: 280px; padding: 10px; border: none; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.html2md__output { height: 280px; padding: 10px; overflow-y: auto; background: var(--bg-surface); color: var(--color-primary); font-family: monospace; font-size: 13px; white-space: pre-wrap; word-break: break-all; }
</style>
