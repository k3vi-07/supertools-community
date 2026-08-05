<template>
  <h-single-layout>
    <div class="md2html">
      <div class="md2html__io">
        <div class="md2html__panel">
          <div class="md2html__header"><span>Markdown 输入</span><button @click="copyHtml">复制 HTML</button></div>
          <textarea v-model="input" class="md2html__ta selectable" spellcheck="false"></textarea>
        </div>
        <div class="md2html__panel">
          <div class="md2html__header"><span>HTML 输出</span></div>
          <div class="md2html__output selectable">{{ output }}</div>
        </div>
      </div>
      <div class="md2html__preview">
        <div class="md2html__header"><span>渲染预览</span></div>
        <div class="md2html__render" v-html="output"></div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref('# Hello World\n\nThis is **bold** and *italic* text.\n\n## List\n\n- Item 1\n- Item 2\n- Item 3\n\n## Code\n\n```js\nconst x = 42;\nconsole.log(x);\n```\n\n## Link\n\nVisit [GitHub](https://github.com)\n\n> This is a blockquote.')

function escapeHtml(s) {
  return s.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
}

const output = computed(() => {
  let html = input.value
  // 转义 HTML
  const codeBlocks = []
  html = html.replace(/```(\w*)\n([\s\S]*?)```/g, (m, lang, code) => {
    const idx = codeBlocks.length
    codeBlocks.push('<pre><code class="language-' + lang + '">' + escapeHtml(code.trim()) + '</code></pre>')
    return '\x00CODEBLOCK' + idx + '\x00'
  })
  // 行内代码
  const inlineCodes = []
  html = html.replace(/`([^`]+)`/g, (m, code) => {
    const idx = inlineCodes.length
    inlineCodes.push('<code>' + escapeHtml(code) + '</code>')
    return '\x00INLINE' + idx + '\x00'
  })
  html = escapeHtml(html)
  // 标题
  html = html.replace(/^###### (.+)$/gm, '<h6>$1</h6>')
  html = html.replace(/^##### (.+)$/gm, '<h5>$1</h5>')
  html = html.replace(/^#### (.+)$/gm, '<h4>$1</h4>')
  html = html.replace(/^### (.+)$/gm, '<h3>$1</h3>')
  html = html.replace(/^## (.+)$/gm, '<h2>$1</h2>')
  html = html.replace(/^# (.+)$/gm, '<h1>$1</h1>')
  // 引用
  html = html.replace(/^&gt; (.+)$/gm, '<blockquote>$1</blockquote>')
  // 粗体斜体
  html = html.replace(/\*\*\*(.+?)\*\*\*/g, '<strong><em>$1</em></strong>')
  html = html.replace(/\*\*(.+?)\*\*/g, '<strong>$1</strong>')
  html = html.replace(/\*(.+?)\*/g, '<em>$1</em>')
  // 链接
  html = html.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2">$1</a>')
  // 图片
  html = html.replace(/!\[([^\]]*)\]\(([^)]+)\)/g, '<img src="$2" alt="$1">')
  // 无序列表
  html = html.replace(/(?:^[-*] .+\n?)+/gm, (m) => {
    const items = m.trim().split('\n').map(l => '<li>' + l.replace(/^[-*] /, '') + '</li>').join('')
    return '<ul>' + items + '</ul>'
  })
  // 有序列表
  html = html.replace(/(?:^\d+\. .+\n?)+/gm, (m) => {
    const items = m.trim().split('\n').map(l => '<li>' + l.replace(/^\d+\. /, '') + '</li>').join('')
    return '<ol>' + items + '</ol>'
  })
  // 水平线
  html = html.replace(/^---$/gm, '<hr>')
  // 段落
  html = html.split('\n\n').map(block => {
    if (block.startsWith('<')) return block
    return '<p>' + block.trim() + '</p>'
  }).join('\n')
  // 还原代码块
  html = html.replace(/\x00CODEBLOCK(\d+)\x00/g, (m, i) => codeBlocks[parseInt(i)])
  html = html.replace(/\x00INLINE(\d+)\x00/g, (m, i) => inlineCodes[parseInt(i)])
  return html
})

function copyHtml() {
  navigator.clipboard?.writeText(output.value)
}
</script>

<style scoped>
.md2html { display: flex; flex-direction: column; gap: 12px; }
.md2html__io { display: flex; gap: 12px; }
.md2html__panel { flex: 1; display: flex; flex-direction: column; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.md2html__header { display: flex; justify-content: space-between; align-items: center; padding: 8px 12px; background: var(--bg-base); border-bottom: 1px solid var(--border-color); font-size: 12px; color: var(--text-tertiary); }
.md2html__header button { padding: 4px 10px; border: 1px solid var(--color-primary); border-radius: 4px; background: transparent; color: var(--color-primary); font-size: 11px; cursor: pointer; }
.md2html__ta { height: 200px; padding: 10px; border: none; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 13px; resize: vertical; outline: none; }
.md2html__output { height: 200px; padding: 10px; overflow-y: auto; background: var(--bg-surface); color: var(--color-primary); font-family: monospace; font-size: 12px; white-space: pre-wrap; word-break: break-all; }
.md2html__preview { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.md2html__render { padding: 16px; color: var(--text-primary); font-size: 14px; line-height: 1.6; }
.md2html__render :deep(h1) { font-size: 24px; font-weight: 700; margin: 8px 0; }
.md2html__render :deep(h2) { font-size: 20px; font-weight: 700; margin: 8px 0; }
.md2html__render :deep(h3) { font-size: 16px; font-weight: 600; margin: 6px 0; }
.md2html__render :deep(code) { padding: 2px 6px; border-radius: 4px; background: var(--bg-base); font-family: monospace; font-size: 13px; color: var(--color-primary); }
.md2html__render :deep(pre) { padding: 12px; border-radius: 8px; background: var(--bg-base); overflow-x: auto; }
.md2html__render :deep(pre code) { padding: 0; background: none; }
.md2html__render :deep(blockquote) { padding-left: 12px; border-left: 3px solid var(--color-primary); color: var(--text-secondary); margin: 8px 0; }
.md2html__render :deep(ul), .md2html__render :deep(ol) { padding-left: 24px; margin: 8px 0; }
.md2html__render :deep(a) { color: var(--color-primary); }
.md2html__render :deep(hr) { border: none; border-top: 1px solid var(--border-color); margin: 12px 0; }
</style>
