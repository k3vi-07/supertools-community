<template>
  <h-single-layout>
    <div class="pinyin">
      <h-multiline-input v-model="input" title="输入中文" placeholder="输入中文汉字..." />
      <div class="pinyin__options">
        <label><input type="radio" v-model="mode" value="full" /> 全拼</label>
        <label><input type="radio" v-model="mode" value="first" /> 首字母</label>
      </div>
      <div class="pinyin__result">
        <div class="pinyin__header">
          <span>拼音结果</span>
          <button v-if="output" class="pinyin__copy" @click="copy(output)">
            <h-icon icon="mdi:content-copy" :size="14" /> 复制
          </button>
        </div>
        <div class="pinyin__output selectable">{{ output || '—' }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const input = ref('你好世界，SuperTools 开发者工具箱！')
const mode = ref<'full' | 'first'>('full')

// 简化版拼音映射（常用字）
const pinyinMap: Record<string, string> = {
  '你':'ni','好':'hao','世':'shi','界':'jie','开':'kai','发':'fa','者':'zhe','工':'gong','具':'ju',
  '箱':'xiang','中':'zhong','文':'wen','人':'ren','大':'da','小':'xiao','多':'duo','少':'shao',
  '上':'shang','下':'xia','左':'zuo','右':'you','前':'qian','后':'hou','里':'li','外':'wai',
  '我':'wo','他':'ta','她':'ta','它':'ta','们':'men','的':'de','是':'shi','有':'you','在':'zai',
  '和':'he','或':'huo','与':'yu','及':'ji','但':'dan','不':'bu','没':'mei','无':'wu','为':'wei',
  '能':'neng','会':'hui','可':'ke','以':'yi','对':'dui','错':'cuo','好':'hao','坏':'huai',
  '天':'tian','地':'di','日':'ri','月':'yue','年':'nian','时':'shi','分':'fen','秒':'miao',
  '电':'dian','脑':'nao','手':'shou','机':'ji','网':'wang','络':'luo','程':'cheng','序':'xu',
  '代':'dai','码':'ma','数':'shu','据':'ju','库':'ku','表':'biao','文':'wen','件':'jian',
  '名':'ming','称':'cheng','类':'lei','型':'xing','值':'zhi','量':'liang','参':'can','数':'shu',
  '测':'ce','试':'shi','调':'diao','试':'shi','运':'yun','行':'xing','停':'ting','止':'zhi',
  '添':'tian','加':'jia','删':'shan','除':'chu','改':'gai','查':'cha','更':'geng','新':'xin',
  '生':'sheng','成':'cheng','转':'zhuan','换':'huan','格':'ge','式':'shi','化':'hua',
  '编':'bian','辑':'ji','复':'fu','制':'zhi','粘':'zhan','贴':'tie','剪':'jian','切':'qie',
  '搜':'sou','索':'suo','过':'guo','滤':'luo','排':'pai','序':'xu','筛':'shai','选':'xuan',
  '配':'pei','置':'zhi','设':'she','置':'zhi','属':'shu','性':'xing','选':'xuan','项':'xiang',
  '帮':'bang','助':'zhu','说':'shuo','明':'ming','注':'zhu','释':'shi','代':'dai','码':'ma',
  '下':'xia','载':'zai','上':'shang','传':'chuan','保':'bao','存':'cun','打':'da','开':'kai',
  '关':'guan','闭':'bi','启':'qi','动':'dong','退':'tui','出':'chu','登':'deng','录':'luo',
  '密':'mi','码':'ma','用':'yong','户':'hu','权':'quan','限':'xian','安':'an','全':'quan',
  '加':'jia','密':'mi','解':'jie','密':'mi','签':'qian','名':'ming','证':'zheng','书':'shu',
  '请':'qing','输':'shu','入':'ru','确':'que','认':'ren','取':'qu','消':'xiao','返':'fan','回':'hui'
}

const output = computed(() => {
  if (!input.value.trim()) return ''
  let result = ''
  for (const char of input.value) {
    const py = pinyinMap[char]
    if (py) {
      result += mode.value === 'first' ? py[0].toUpperCase() : py + ' '
    } else {
      result += char
    }
  }
  return result.trim()
})

function copy(text: string): void {
  window.$he3?.copyText(text)
  window.$he3?.message.success('已复制')
}
</script>

<style scoped>
.pinyin { display: flex; flex-direction: column; gap: 16px; }
.pinyin__options { display: flex; gap: 16px; }
.pinyin__options label { display: flex; align-items: center; gap: 4px; font-size: 13px; color: var(--text-secondary); cursor: pointer; }
.pinyin__result { border: 1px solid var(--border-color); border-radius: 8px; overflow: hidden; }
.pinyin__header { display: flex; align-items: center; justify-content: space-between; padding: 8px 12px; background: var(--bg-code-header); border-bottom: 1px solid var(--border-color); font-size: 12px; font-weight: 600; color: var(--text-secondary); }
.pinyin__copy { display: flex; align-items: center; gap: 4px; padding: 2px 8px; border: 1px solid var(--border-color); border-radius: 4px; background: transparent; color: var(--text-secondary); font-size: 11px; cursor: pointer; }
.pinyin__copy:hover { background: var(--bg-hover); }
.pinyin__output { padding: 16px; font-size: 16px; color: var(--color-primary); word-break: break-all; }
</style>
