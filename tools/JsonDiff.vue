<template>
  <h-single-layout>
    <div class="json-min">
      <div class="json-min__io">
        <div class="json-min__panel">
          <div class="json-min__header"><span>JSON 1</span><button @click="paste1">粘贴</button></div>
          <textarea v-model="json1" class="json-min__ta selectable" spellcheck="false"></textarea>
        </div>
        <div class="json-min__panel">
          <div class="json-min__header"><span>JSON 2</span><button @click="paste2">粘贴</button></div>
          <textarea v-model="json2" class="json-min__ta selectable" spellcheck="false"></textarea>
        </div>
      </div>
      <div class="json-min__result">
        <div class="json-min__result-header"><span>差异</span></div>
        <div class="json-min__diff-list">
          <div v-for="d in diffs" :key="d.path" class="json-min__diff-item" :class="d.type">
            <span class="json-min__path">{{ d.path }}</span>
            <span class="json-min__type">{{ d.type === 'added' ? '+ 新增' : d.type === 'removed' ? '- 删除' : '~ 修改' }}</span>
            <code v-if="d.val1 !== undefined" class="json-min__val">{{ d.val1 }}</code>
            <code v-if="d.val2 !== undefined" class="json-min__val">{{ d.val2 }}</code>
          </div>
          <div v-if="diffs.length === 0 && json1 && json2" class="json-min__same">✅ 两个 JSON 完全相同</div>
        </div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
const json1 = ref('{\n  "name": "Alice",\n  "age": 30,\n  "city": "Beijing",\n  "skills": ["JS", "Vue"]\n}')
const json2 = ref('{\n  "name": "Alice",\n  "age": 25,\n  "city": "Shanghai",\n  "skills": ["JS", "Vue", "TS"]\n}')

async function paste1(): Promise<void> { json1.value = await navigator.clipboard.readText() }
async function paste2(): Promise<void> { json2.value = await navigator.clipboard.readText() }

interface Diff { path: string; type: string; val1?: unknown; val2?: unknown }

const diffs = computed<Diff[]>(() => {
  try {
    const a = JSON.parse(json1.value)
    const b = JSON.parse(json2.value)
    const result: Diff[] = []
    compare(a, b, '', result)
    return result
  } catch { return [] }
})

function compare(a: unknown, b: unknown, path: string, result: Diff[]): void {
  if (a === b) return
  if (typeof a !== typeof b) {
    result.push({ path: path || 'root', type: 'changed', val1: a, val2: b })
    return
  }
  if (typeof a !== 'object' || a === null || b === null) {
    if (a !== b) result.push({ path: path || 'root', type: 'changed', val1: a, val2: b })
    return
  }
  if (Array.isArray(a) && Array.isArray(b)) {
    const max = Math.max(a.length, b.length)
    for (let i = 0; i < max; i++) {
      if (i >= a.length) result.push({ path: `${path}[${i}]`, type: 'added', val2: b[i] })
      else if (i >= b.length) result.push({ path: `${path}[${i}]`, type: 'removed', val1: a[i] })
      else compare(a[i], b[i], `${path}[${i}]`, result)
    }
    return
  }
  const allKeys = new Set([...Object.keys(a as object), ...Object.keys(b as object)])
  for (const key of allKeys) {
    const va = (a as Record<string, unknown>)[key]
    const vb = (b as Record<string, unknown>)[key]
    const p = path ? `${path}.${key}` : key
    if (va === undefined) result.push({ path: p, type: 'added', val2: vb })
    else if (vb === undefined) result.push({ path: p, type: 'removed', val1: va })
    else compare(va, vb, p, result)
  }
}
</script>

<style scoped>
.json-min { display: flex; flex-direction: column; gap: 16px; }
.json-min__io { display: flex; gap: 12px; }
.json-min__panel { flex: 1; display: flex; flex-direction: column; border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.json-min__header { display: flex; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; }
.json-min__header button { border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; padding: 2px 8px; }
.json-min__ta { height: 120px; padding: 10px; border: none; background: var(--bg-code); color: var(--text-primary); font-family: monospace; font-size: 12px; resize: vertical; outline: none; }
.json-min__result { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.json-min__result-header { padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; font-weight: 600; }
.json-min__diff-list { padding: 8px; max-height: 300px; overflow-y: auto; }
.json-min__diff-item { display: flex; align-items: center; gap: 8px; padding: 6px 10px; border-radius: 4px; margin-bottom: 4px; font-size: 12px; }
.json-min__diff-item.added { background: color-mix(in srgb, var(--color-success) 15%, transparent); }
.json-min__diff-item.removed { background: color-mix(in srgb, var(--color-error) 15%, transparent); }
.json-min__diff-item.changed { background: color-mix(in srgb, var(--color-warning) 15%, transparent); }
.json-min__path { flex: 1; font-family: monospace; color: var(--text-primary); }
.json-min__type { font-size: 11px; color: var(--text-tertiary); }
.json-min__val { font-family: monospace; color: var(--color-primary); }
.json-min__same { text-align: center; padding: 20px; color: var(--color-success); }
</style>
