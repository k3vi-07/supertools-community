<template>
  <h-single-layout>
    <div class="sqli-check">
      <textarea v-model="input" class="sqli-check__input selectable" placeholder="输入要检测的 SQL 语句或用户输入..." spellcheck="false"></textarea>

      <div v-if="input.trim()" class="sqli-check__result">
        <div class="sqli-check__header">
          <span>风险等级:</span>
          <span class="sqli-check__risk" :class="riskLevel">{{ riskLabel }}</span>
        </div>
        <div class="sqli-check__score-bar">
          <div class="sqli-check__score-fill" :class="riskLevel" :style="{ width: Math.min(100, riskScore * 20) + '%' }"></div>
        </div>
        <div v-if="matches.length > 0" class="sqli-check__matches">
          <div v-for="m in matches" :key="m.pattern" class="sqli-check__match">
            <span class="sqli-check__match-pattern">{{ m.matched }}</span>
            <span class="sqli-check__match-desc">{{ m.desc }}</span>
            <span class="sqli-check__match-severity" :class="'sev-' + m.severity">{{ severityLabel(m.severity) }}</span>
          </div>
        </div>
        <div v-else class="sqli-check__safe">✅ 未检测到明显的 SQL 注入特征</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'

const input = ref("1' OR '1'='1")

// SQL 注入特征模式
const patterns = [
  { regex: /(\bOR\b|\bAND\b)\s+['"]?\d+['"]?\s*=\s*['"]?\d+/i, desc: '布尔条件恒真 (OR 1=1)', severity: 3 },
  { regex: /'\s*(OR|AND)\s*'/i, desc: '引号闭合 + 逻辑运算符', severity: 3 },
  { regex: /--|#|\/\*/, desc: 'SQL 注释符号 (--, #, /*)', severity: 2 },
  { regex: /;\s*(DROP|DELETE|UPDATE|INSERT|ALTER|CREATE|TRUNCATE)\s/i, desc: '语句拼接 + 危险操作', severity: 3 },
  { regex: /UNION\s+(ALL\s+)?SELECT/i, desc: 'UNION SELECT 联合查询注入', severity: 3 },
  { regex: /\b(DROP|TRUNCATE)\s+TABLE/i, desc: '删除表操作', severity: 3 },
  { regex: /INTO\s+(OUT|DUMP)FILE/i, desc: '文件写入注入', severity: 3 },
  { regex: /xp_cmdshell|sp_executesql/i, desc: '存储过程执行', severity: 3 },
  { regex: /\bWAITFOR\s+DELAY\b/i, desc: '时间盲注 (WAITFOR DELAY)', severity: 2 },
  { regex: /SLEEP\s*\(/i, desc: '时间盲注 (SLEEP)', severity: 2 },
  { regex: /BENCHMARK\s*\(/i, desc: '时间盲注 (BENCHMARK)', severity: 2 },
  { regex: /LOAD_FILE\s*\(/i, desc: '文件读取注入', severity: 3 },
  { regex: /CONCAT\s*\(|GROUP_CONCAT\s*\(/i, desc: '字符串拼接函数（常用于注入提取数据）', severity: 1 },
  { regex: /INFORMATION_SCHEMA/i, desc: '访问系统表 INFORMATION_SCHEMA', severity: 2 },
  { regex: /\bEXEC(UTE)?\s*\(|\bEXEC(UTE)?\s/i, desc: '动态 SQL 执行', severity: 2 },
  { regex: /'\s*;\s*--/, desc: '引号 + 分号 + 注释（经典注入结尾）', severity: 3 },
  { regex: /0x[0-9a-f]+/i, desc: '十六进制编码（可能绕过 WAF）', severity: 1 }
]

const matches = computed(() => {
  const results = []
  for (const p of patterns) {
    const m = input.value.match(p.regex)
    if (m) {
      results.push({ pattern: p.regex.source, matched: m[0].substring(0, 60), desc: p.desc, severity: p.severity })
    }
  }
  return results
})

const riskScore = computed(() => {
  if (matches.value.length === 0) return 0
  return matches.value.reduce((sum, m) => sum + m.severity, 0)
})

const riskLevel = computed(() => {
  if (riskScore.value === 0) return 'safe'
  if (riskScore.value <= 2) return 'low'
  if (riskScore.value <= 5) return 'medium'
  return 'high'
})

const riskLabel = computed(() => {
  const labels = { safe: '安全', low: '低风险', medium: '中风险', high: '高风险' }
  return labels[riskLevel.value] || '未知'
})

function severityLabel(s) {
  return ['', '提示', '警告', '危险'][s] || '未知'
}
</script>

<style scoped>
.sqli-check { display: flex; flex-direction: column; gap: 16px; }
.sqli-check__input { width: 100%; min-height: 100px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-family: monospace; font-size: 14px; resize: vertical; outline: none; }
.sqli-check__result { display: flex; flex-direction: column; gap: 12px; }
.sqli-check__header { display: flex; align-items: center; gap: 8px; font-size: 14px; color: var(--text-secondary); }
.sqli-check__risk { font-size: 18px; font-weight: 700; }
.sqli-check__risk.safe { color: #22c55e; }
.sqli-check__risk.low { color: #3b82f6; }
.sqli-check__risk.medium { color: #f59e0b; }
.sqli-check__risk.high { color: #ef4444; }
.sqli-check__score-bar { height: 8px; border-radius: 4px; background: var(--bg-base); overflow: hidden; }
.sqli-check__score-fill { height: 100%; border-radius: 4px; transition: width 0.3s; }
.sqli-check__score-fill.safe { background: #22c55e; }
.sqli-check__score-fill.low { background: #3b82f6; }
.sqli-check__score-fill.medium { background: #f59e0b; }
.sqli-check__score-fill.high { background: #ef4444; }
.sqli-check__matches { display: flex; flex-direction: column; gap: 6px; }
.sqli-check__match { display: flex; align-items: center; gap: 10px; padding: 8px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); }
.sqli-check__match-pattern { font-family: monospace; font-size: 13px; color: var(--color-primary); flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.sqli-check__match-desc { font-size: 12px; color: var(--text-secondary); flex: 2; }
.sqli-check__match-severity { font-size: 11px; padding: 2px 8px; border-radius: 10px; flex-shrink: 0; }
.sqli-check__match-severity.sev-1 { background: rgba(59,130,246,0.15); color: #3b82f6; }
.sqli-check__match-severity.sev-2 { background: rgba(245,158,11,0.15); color: #f59e0b; }
.sqli-check__match-severity.sev-3 { background: rgba(239,68,68,0.15); color: #ef4444; }
.sqli-check__safe { padding: 16px; text-align: center; color: #22c55e; font-size: 14px; }
</style>
