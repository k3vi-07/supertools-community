<div align="center">

# 🧩 SuperTools 社区工具仓库

> 为 [SuperTools](https://github.com/k3vi-07/supertools) 开发者工具箱提供 **93 个**远程工具插件

[![Tool Count](https://img.shields.io/badge/Tools-93-7c3aed)](https://github.com/k3vi-07/supertools-community)
[![Version](https://img.shields.io/badge/Version-v1.7.0-success)](https://github.com/k3vi-07/supertools-community/releases)
[![License](https://img.shields.io/badge/License-MIT-blue)](LICENSE)
[![CDN](https://img.shields.io/badge/CDN-jsDelivr-orange)](https://cdn.jsdelivr.net/gh/k3vi-07/supertools-community/)

通过 GitHub + jsDelivr CDN 动态加载，无需更新应用即可使用新工具。

</div>

---

## 🚀 快速开始

在 SuperTools 应用的 **工具商店** 中输入：

```
k3vi-07/supertools-community
```

点击 **添加** → 浏览工具 → **安装**。安装的工具会自动出现在搜索和分类中。

> 💡 SuperTools v1.4.0+ 首次使用会自动添加本仓库，无需手动输入。

## 📦 全部工具（93 个）

### 🔐 加密哈希（43 个）

<details>
<summary>点击展开全部 43 个加密工具</summary>

#### 对称加密（12）

| 工具 | 说明 |
|------|------|
| AES-CBC 加密 | AES-256-CBC + PKCS7 填充 |
| AES-CTR 加密 | AES-256-CTR 计数器模式流加密 |
| AES-GCM 认证加密 | AES-256-GCM 认证加密（AEAD） |
| Blowfish 加密 | Blowfish 64 位分组密码，可变长密钥 |
| Camellia 加密 | Camellia 国际标准分组密码 |
| DES/3DES 加解密 | DES/3DES 对称加密（ECB/CBC） |
| IDEA 加密 | IDEA 国际数据加密算法 |
| RC4 流密码 | RC4 流密码加解密 |
| SM4 国密加密 | SM4 国密分组密码（ECB 模式） |
| Salsa20 流密码 | Daniel Bernstein 设计的高速流密码 |
| TEA 加密 | Tiny Encryption Algorithm 轻量级密码 |
| Twofish 加密 | AES 候选算法，128 位分组 |
| XTEA/XXTEA 加密 | XTEA 和 XXTEA 分组密码 |

#### 哈希算法（14）

| 工具 | 说明 |
|------|------|
| BLAKE2 哈希 | BLAKE2s 快速加密哈希（256/224/160 位） |
| BCrypt 验证器 | 解析并验证 BCrypt 密码哈希 |
| CityHash 哈希 | Google CityHash 非加密哈希（64/128 位） |
| FNV 哈希 | FNV-1 / FNV-1a 哈希（32/64 位） |
| GOST 哈希 | GOST R 34.11-94 俄罗斯国标哈希 |
| HMAC 全系列 | HMAC（SHA-1/256/384/512）实时计算 |
| Murmur 哈希 | MurmurHash2 / MurmurHash3 |
| SM3 国密哈希 | 中国国密杂凑算法（GM/T 0004） |
| SipHash 哈希 | SipHash-2-4 / SipHash-1-3 PRF 哈希 |
| Snefru 哈希 | Snefru 256 位哈希（Merkle 设计） |
| Tiger 哈希 | Tiger 192 位哈希，64 位优化 |
| Whirlpool 哈希 | Whirlpool 512 位（ISO/IEC 10118-3） |
| 哈希值识别 | 识别哈希类型和可能的加密算法 |
| 文件哈希计算 | 计算文件的 SHA-1/256/384/512 + MD5 |

#### 密钥派生 & 签名（5）

| 工具 | 说明 |
|------|------|
| PBKDF2 密钥派生 | 迭代/盐/长度可调 |
| scrypt 密钥派生 | 内存困难型密钥派生（N/r/p 参数） |
| Argon2 密钥派生 | Argon2 参数展示 + PBKDF2 |
| ECDSA 数字签名 | 密钥对生成、签名、验签 |
| ECDH 密钥交换 | 椭圆曲线密钥交换演示 |

#### 安全工具（5）

| 工具 | 说明 |
|------|------|
| 密码强度检测 | 检测密码强度、熵值和破解时间 |
| CSP 生成器 | Content-Security-Policy 头生成器 |
| SQL 注入检测 | SQL 注入风险模式检测 |
| XSS 净化器 | 检测和净化 XSS 攻击代码 |
| 熵值计算器 | 计算 Shannon 熵值和字符分布 |

#### 经典密码 & 其他（4）

| 工具 | 说明 |
|------|------|
| 经典密码合集 | 凯撒/ROT13/维吉尼亚/Atbash/仿射/栅栏 |
| Playfair 密码 | 5×5 双字母替换密码 |
| JWT 生成器 | 使用密钥生成 JWT Token |
| 文本加密器 | XOR 加密 + Base64 编码 |

</details>

### 📝 文本工具（15 个）

| 工具 | 说明 |
|------|------|
| 文本转语音 | 使用 Web Speech API 朗读文本 |
| 数字转中文大写 | 阿拉伯数字转人民币大写金额 |
| 中文转拼音 | 汉字转全拼/首字母 |
| Emoji 表情选择器 | 搜索和复制 Emoji 表情符号 |
| ASCII 艺术字 | 文本转 ASCII Art 字符画 |
| 特殊符号选择器 | 数学、箭头、花色等特殊符号 |
| 字符计数器 | 实时统计字符/单词/行数/中文字数 |
| 趣味文字翻转 | 上下翻转/镜像/气泡字/全角 |
| 随机字符串生成 | crypto 安全随机字符串 |
| 文本行去重 | 保留首次出现顺序的去重 |
| 文本排序 | 升序/降序/自然/随机排序 |
| 多行文本反转 | 字符/行序/单词反转 4 种模式 |
| 批量大小写转换 | 12 种命名风格一键转换 |
| Markdown 转 HTML | Markdown 转 HTML 代码 |
| HTML 转 Markdown | HTML 转 Markdown 格式 |

### 🌐 前端开发（10 个）

| 工具 | 说明 |
|------|------|
| 单位换算器 | 长度/重量/温度/面积/速度/数据 |
| 房贷计算器 | 等额本息月供计算 |
| 复利计算器 | 投资收益 + 72 法则翻倍时间 |
| UTM 链接生成器 | 生成营销追踪 UTM 参数链接 |
| Grid 布局生成器 | 可视化 CSS Grid 布局生成 |
| HTML 符号速查 | HTML 特殊符号实体编码 |
| 汇率换算 | 10 种货币实时汇率 |
| CSS 单位转换器 | px/rem/em/pt/vw/vh 相互转换 |
| HTTP 状态码速查 | HTTP 状态码分类速查表 |
| MIME 类型查询 | 文件扩展名与 MIME 类型互查 |

### 💻 编程工具（8 个）

| 工具 | 说明 |
|------|------|
| Chmod 权限计算器 | Linux 文件权限可视化计算 |
| 键盘 KeyCode 查询 | 实时查询键盘按键 keyCode |
| 进制转换器 | 二/八/十/十六进制互转 |
| 浮点数转换 | IEEE 754 与十六进制互转 |
| 正则表达式测试 | 实时测试正则匹配结果 |
| 正则表达式速查 | 正则语法速查表 |
| UUID 多版本生成 | v4 / nil，支持多种格式 |
| Cron 表达式生成器 | 交互式生成 + 预设模式 |

### 🔗 编码解码（6 个）

| 工具 | 说明 |
|------|------|
| 二维码识别 | 上传图片识别二维码内容 |
| 条形码生成器 | 生成 CODE128/CODE39 条形码 |
| 文本转义工具 | HTML/URL/Unicode/Hex/JS 转义 |
| Base64 图片互转 | 图片与 Base64 互转，拖拽上传 |
| Hex/Base64 互转 | Hex 与 Base64 双向转换 |
| PEM 格式化 | 证书格式化、去换行、提取 Base64 |

### 📋 JSON 工具（3 个）

| 工具 | 说明 |
|------|------|
| JSON 转 XML | JSON 数据转 XML 格式 |
| JSON 对比 | 深度对比两个 JSON 的差异 |
| JSON Schema 生成器 | 从 JSON 数据自动推断 Schema |

### ⏰ 时间日期（3 个）

| 工具 | 说明 |
|------|------|
| 倒计时器 | 实时倒计时到指定日期 |
| 秒表 | 高精度计时，支持记圈 |
| Cron 表达式生成器 Pro | 可视化生成，9 种预设 |

### 🌍 网络工具（3 个）

| 工具 | 说明 |
|------|------|
| IP 定位 | IP 地理位置查询 + 地图 |
| Ping 延迟测试 | 网站延迟检测 |
| HTTP 请求测试器 | GET/POST/PUT/DELETE，显示状态和耗时 |

### 🎨 颜色工具（2 个）

| 工具 | 说明 |
|------|------|
| 颜色对比度检查 | WCAG AA/AAA 合规检查 |
| 颜色提取器 | 从 CSS/HTML 代码提取所有颜色 |

## 🏗️ 技术架构

```
GitHub 仓库 (.vue 源码 + registry.json)
       ↓
jsDelivr CDN (全球加速 + CORS)
       ↓
Electron 主进程 (net.fetch 代理，域名白名单 + 大小限制)
       ↓ IPC
渲染进程 (vue3-sfc-loader 运行时编译)
       ↓
Vue 组件渲染 (defineAsyncComponent + Suspense)
```

## 📋 registry.json 格式

```json
{
  "name": "仓库名称",
  "version": "v1.0.0",
  "tools": [
    {
      "id": "tool-id",
      "name": "English Name",
      "nameZh": "中文名",
      "icon": "mdi:icon-name",
      "category": ["text"],
      "keywords": ["关键词"],
      "description": "工具描述",
      "path": "tools/Tool.vue",
      "author": "your-name",
      "version": "v1.0.0"
    }
  ]
}
```

**合法分类：** `encode` · `json` · `cryptography` · `text` · `web` · `color` · `datetime` · `programming` · `network`

## 🛠️ 创建自己的工具

1. **Fork** 本仓库或创建新仓库
2. 在 `tools/` 目录创建 `.vue` 组件（标准 Vue 3 SFC）
3. 在 `registry.json` 中注册工具
4. 推送到 GitHub
5. 在 SuperTools 中添加你的仓库地址

### 可用组件

| 组件 | 说明 |
|------|------|
| `<h-single-layout>` | 单栏居中布局容器 |
| `<h-transform>` | 双栏输入→输出转换 |
| `<h-text-transform>` | 纯文本转换 |
| `<h-code-editor>` | 代码编辑器 |
| `<h-button>` / `<h-input>` / `<h-select>` / `<h-switch>` | 表单组件 |
| `<h-icon>` | Iconify 图标（mdi: 前缀） |

### 可用 API

```javascript
window.$he3.copyText('文本')              // 复制到剪贴板
window.$he3.message.success('成功')       // 消息提示
window.$he3.message.error('失败')
window.$he3.shellOpenExternal('https://...') // 打开浏览器
```

### ⚠️ 注意事项

- **不能 import npm 包**：远程工具在浏览器端运行时编译，请使用浏览器原生 API（如 [Web Crypto API](https://developer.mozilla.org/docs/Web/API/Web_Crypto_API) 替代 crypto-js）
- **使用 CSS 变量**适配深色/浅色模式：`var(--bg-surface)`、`var(--text-primary)` 等
- **图标**从 [Material Design Icons](https://pictogrammers.com/library/mdi/) 查找，使用 `mdi:` 前缀

## 📜 License

MIT
