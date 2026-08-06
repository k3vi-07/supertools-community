<template>
  <h-single-layout>
    <h-transform
      left-title="JSON 数据"
      right-title="JSON Schema"
      :sample-data="sample"
      :input-handler="generate"
    />
  </h-single-layout>
</template>

<script setup>
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

function inferType(val) {
  if (val === null) return 'null'
  if (Array.isArray(val)) return 'array'
  return typeof val
}

function generateSchema(data) {
  const schema = { type: inferType(data) }

  if (schema.type === 'object') {
    schema.properties = {}
    const keys = Object.keys(data)
    if (keys.length > 0) {
      schema.required = keys
    }
    for (const key of keys) {
      schema.properties[key] = generateSchema(data[key])
    }
  } else if (schema.type === 'array') {
    if (data.length > 0) {
      schema.items = generateSchema(data[0])
    } else {
      schema.items = {}
    }
  } else if (schema.type !== 'null') {
    if (schema.type === 'string') {
      schema.example = data
    }
  }

  return schema
}

function generate(input) {
  try {
    const data = JSON.parse(input)
    const schema = generateSchema(data)
    return JSON.stringify(schema, null, 2)
  } catch (err) {
    return '❌ JSON 解析失败: ' + err.message
  }
}
</script>
