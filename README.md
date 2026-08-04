# 🛠️ SuperTools 社区工具仓库

> 为 [SuperTools](https://github.com/k3vi-07/supertools) 开发者工具箱提供 **31 个**远程工具插件，通过 GitHub + jsDelivr CDN 动态加载。

[![Tool Count](https://img.shields.io/badge/Tools-31-7c3aed)](https://github.com/k3vi-07/supertools-community)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![CDN](https://img.shields.io/badge/CDN-jsDelivr-orange)](https://cdn.jsdelivr.net/gh/k3vi-07/supertools-community/)

## 🚀 快速开始

在 SuperTools 应用的 **工具商店** 中输入：

```
k3vi-07/supertools-community
```

点击 **添加** → 浏览工具 → **安装**。安装的工具会自动出现在搜索和分类中。

## 📦 全部工具（31 个）

### 📝 文本工具（9 个）

| 工具 | 说明 |
|------|------|
| 🔊 文本转语音 | 使用 Web Speech API 朗读文本 |
| 💰 数字转中文大写 | 阿拉伯数字转人民币大写金额 |
| 🔤 中文转拼音 | 汉字转全拼/首字母 |
| 😀 Emoji 选择器 | 分类浏览和搜索 Emoji 表情 |
| 🎨 ASCII 艺术字 | 文本转 ASCII Art 字符画 |
| ⚛️ 特殊符号选择器 | 数学、箭头、花色等特殊符号 |
| 📝 字符计数器 | 实时统计字符/单词/行数/中文字数 |
| 🔄 趣味文字翻转 | 上下翻转/镜像/气泡字/全角宽字 |
| 🔀 随机字符串生成 | crypto 安全随机字符串 |

### 🌐 前端开发（7 个）

| 工具 | 说明 |
|------|------|
| 📏 单位换算器 | 长度/重量/温度/面积/速度/数据 |
| 🏠 房贷计算器 | 等额本息月供计算 |
| 📈 复利计算器 | 投资收益 + 72 法则翻倍时间 |
| 🔗 UTM 链接生成器 | 生成营销追踪 UTM 参数链接 |
| 🔲 Grid 布局生成器 | 可视化 CSS Grid 布局生成 |
| 📋 HTML 符号速查 | 60+ HTML 特殊符号实体编码 |
| 💱 汇率换算 | 10 种货币实时汇率 |

### 💻 编程工具（4 个）

| 工具 | 说明 |
|------|------|
| 🔒 Chmod 计算器 | Linux 文件权限可视化计算 |
| ⌨️ KeyCode 查询 | 实时查询键盘按键 keyCode |
| 🔢 进制转换器 | 二/八/十/十六进制互转 |
| 📊 浮点数转换 | IEEE 754 与十六进制互转 |

### 🔐 加密工具（2 个）

| 工具 | 说明 |
|------|------|
| 🔍 哈希值识别 | 识别哈希类型和可能的加密算法 |
| 🔑 JWT 生成器 | 使用 HMAC-SHA256 签名生成 JWT |

### 🔗 编码工具（2 个）

| 工具 | 说明 |
|------|------|
| 📷 二维码识别 | 上传图片识别二维码内容 |
| 🏷️ 条形码生成器 | 生成 CODE128/CODE39 条形码 |

### 📋 JSON 工具（2 个）

| 工具 | 说明 |
|------|------|
| 🔄 JSON 转 XML | JSON 数据转 XML 格式 |
| 🔍 JSON 对比 | 深度对比两个 JSON 的差异 |

### ⏰ 时间工具（2 个）

| 工具 | 说明 |
|------|------|
| ⏰ 倒计时器 | 实时倒计时到指定日期 |
| ⏱️ 秒表 | 高精度计时，支持记圈 |

### 🌍 网络工具（2 个）

| 工具 | 说明 |
|------|------|
| 📍 IP 定位 | IP 地理位置查询 + 地图 |
| 📡 Ping 延迟测试 | 网站延迟检测 |

### 🎨 颜色工具（1 个）

| 工具 | 说明 |
|------|------|
| 🎨 颜色对比度检查 | WCAG AA/AAA 合规检查 |

## 🏗️ 技术架构

```
GitHub 仓库 (.vue 源码 + registry.json)
       ↓
jsDelivr CDN (全球加速 + CORS)
       ↓
Electron 主进程 (net.fetch 代理，绕过 CORS)
       ↓ IPC
渲染进程 (vue3-sfc-loader 运行时编译)
       ↓
Vue 组件渲染 (defineAsyncComponent + Suspense)
```

## 📋 registry.json 格式

```json
{
  "name": "仓库名称",
  "tools": [
    {
      "id": "tool-id",           // 唯一标识 (kebab-case)
      "name": "English Name",    // 英文名
      "nameZh": "中文名",         // 中文名
      "icon": "mdi:icon-name",   // Iconify 图标
      "category": ["text"],      // 分类
      "keywords": ["关键词"],     // 搜索关键词
      "description": "描述",      // 工具说明
      "path": "tools/Tool.vue",  // 组件路径
      "author": "author",        // 作者 (可选)
      "version": "1.0.0"         // 版本 (可选)
    }
  ]
}
```

## 🛠️ 创建自己的工具

1. **Fork** 本仓库或创建新仓库
2. 在 `tools/` 目录创建 `.vue` 组件（使用标准 Vue 3 SFC）
3. 在 `registry.json` 中注册工具
4. 推送到 GitHub
5. 在 SuperTools 中添加你的仓库地址

### 工具组件示例

```vue
<template>
  <h-single-layout>
    <h-text-transform :transform="myFn" />
  </h-single-layout>
</template>

<script setup lang="ts">
function myFn(input: string): string {
  return input.toUpperCase()
}
</script>
```

> 💡 工具可使用所有 SuperTools 内置的 `h-` 前缀全局组件（`h-single-layout`、`h-transform`、`h-text-transform` 等）和 `window.$he3` API。

## 📜 License

MIT
