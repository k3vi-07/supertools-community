# SuperTools 社区工具仓库

为 [SuperTools](https://github.com/k3vi-07/supertools) 开发者工具箱提供远程工具插件。

## 📦 包含的工具

| 工具 | 说明 |
|------|------|
| JSON 转 XML | 将 JSON 数据转换为 XML 格式 |
| 文本转语音 | 使用 Web Speech API 朗读文本 |
| 数字转中文大写 | 阿拉伯数字转中文大写金额 |
| 中文转拼音 | 汉字转全拼/首字母 |
| Emoji 表情选择器 | 搜索和复制 Emoji |

## 🔗 在 SuperTools 中使用

在 SuperTools 的 **工具商店** 中输入仓库地址：

```
k3vi-07/supertools-community
```

然后点击添加，即可浏览和安装本仓库中的所有工具。

## 🛠️ 创建自己的远程工具

### 1. Fork 本仓库或创建新仓库

### 2. 添加工具组件

在 `tools/` 目录创建 `.vue` 文件：

```vue
<template>
  <h-single-layout>
    <h-text-transform :transform="myTransform" />
  </h-single-layout>
</template>

<script setup lang="ts">
function myTransform(input: string): string {
  return input.toUpperCase()
}
</script>
```

### 3. 在 registry.json 中注册

```json
{
  "tools": [
    {
      "id": "my-tool",
      "name": "My Tool",
      "nameZh": "我的工具",
      "icon": "mdi:tools",
      "category": ["text"],
      "keywords": ["my", "tool"],
      "description": "工具描述",
      "path": "tools/MyTool.vue"
    }
  ]
}
```

### 4. 推送到 GitHub

工具会自动通过 jsDelivr CDN 分发。

## 📋 registry.json 格式

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 唯一标识符 |
| `name` | string | 英文名 |
| `nameZh` | string | 中文名 |
| `icon` | string | Iconify 图标名 |
| `category` | string[] | 分类 |
| `keywords` | string[] | 搜索关键词 |
| `description` | string | 描述 |
| `path` | string | 组件文件路径 |
| `author` | string? | 作者 |
| `version` | string? | 版本 |

## License

MIT
