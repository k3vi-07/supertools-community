<template>
  <h-single-layout>
    <h-transform
      left-title="JSON 输入"
      right-title="XML 输出"
      input-lang="json"
      output-lang="xml"
      :sample-data="sample"
      :input-handler="convertFn"
    />
  </h-single-layout>
</template>

<script setup lang="ts">
const sample = `{
  "name": "SuperTools",
  "version": "1.0.0",
  "isOpenSource": true,
  "tags": ["electron", "vue"],
  "author": {
    "name": "Dev",
    "email": "dev@example.com"
  }
}`

function convertFn(input: string): string {
  try {
    const data = JSON.parse(input)
    return '<?xml version="1.0" encoding="UTF-8"?>\n' + jsonToXml(data, 'root', 0)
  } catch (err) {
    return `❌ ${(err as Error).message}`
  }
}

function jsonToXml(data: unknown, tagName: string, indent: number): string {
  const pad = '  '.repeat(indent)
  if (data === null || data === undefined) return `${pad}<${tagName}></${tagName}>`
  if (typeof data !== 'object') return `${pad}<${tagName}>${escapeXml(String(data))}</${tagName}>`
  if (Array.isArray(data)) {
    return data.map((item) => jsonToXml(item, tagName.replace(/s$/, ''), indent)).join('\n')
  }
  const entries = Object.entries(data as Record<string, unknown>)
  const inner = entries.map(([key, val]) => jsonToXml(val, key, indent + 1)).join('\n')
  return `${pad}<${tagName}>\n${inner}\n${pad}</${tagName}>`
}

function escapeXml(s: string): string {
  return s.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;').replace(/'/g, '&apos;')
}
</script>
