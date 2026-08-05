<template>
  <h-single-layout>
    <div class="flip">
      <div class="flip__controls">
        <button v-for="t in types" :key="t.id" :class="{active: type===t.id}" @click="type=t.id">{{ t.label }}</button>
      </div>
      <div class="flip__io">
        <textarea v-model="input" class="flip__input selectable" :placeholder="placeholder" spellcheck="false" @input="convert"></textarea>
        <div class="flip__output selectable">{{ output }}</div>
      </div>
    </div>
  </h-single-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
const type = ref('upside')
const types = [
  { id: 'upside', label: '上下翻转' },
  { id: 'mirror', label: '左右镜像' },
  { id: 'strike', label: '删除线' },
  { id: 'bubble', label: '气泡字' },
  { id: 'wide', label: '全角宽字' }
]

const upsideMap: Record<string,string> = {'a':'ɐ','b':'q','c':'ɔ','d':'p','e':'ǝ','f':'ɟ','g':'ƃ','h':'ɥ','i':'ᴉ','j':'ɾ','k':'ʞ','l':'l','m':'ɯ','n':'u','o':'o','p':'d','q':'b','r':'ɹ','s':'s','t':'ʇ','u':'n','v':'ʌ','w':'ʍ','x':'x','y':'ʎ','z':'z','A':'∀','B':'𐐒','C','Ɔ','D','ᗡ','E','Ǝ','F','Ⅎ','G','⅁','H','H','I','I','J','ſ','K','ʞ','L','˥','M','W','N','N','O','O','P','Ԁ','Q','Ò','R','ɹ','S','S','T','┴','U','∩','V','Λ','W','M','X','X','Y','⅄','Z','Z'}
const mirrorMap: Record<string,string> = {'a':'ɒ','b':'d','c':'ɔ','d':'b','e':'ɘ','f':'Ꮈ','g':'ǫ','h':'ʜ','i':'i','j':'ꞁ','k':'ʞ','l':'l','m':'m','n':'n','o':'o','p':'q','q':'p','r':'ɿ','s':'ꙅ','t':'ƚ','u':'u','v':'v','w':'w','x':'x','y':'y','z':'z'}
const bubbleMap: Record<string,string> = {'a':'ⓐ','b':'ⓑ','c':'ⓒ','d':'ⓓ','e':'ⓔ','f':'ⓕ','g':'ⓖ','h':'ⓗ','i':'ⓘ','j':'ⓙ','k':'ⓚ','l':'ⓛ','m':'ⓜ','n':'ⓝ','o':'ⓞ','p':'ⓟ','q':'ⓠ','r':'ⓡ','s':'ⓢ','t':'ⓣ','u':'ⓤ','v':'ⓥ','w':'ⓦ','x':'ⓧ','y':'ⓨ','z':'ⓩ','A':'Ⓐ','B':'Ⓑ','C':'Ⓒ','D':'Ⓓ','E':'Ⓔ','F':'Ⓕ','G':'Ⓖ','H':'Ⓗ','I':'Ⓘ','J':'Ⓙ','K':'Ⓚ','L':'Ⓛ','M':'Ⓜ','N':'Ⓝ','O':'Ⓞ','P':'Ⓟ','Q':'Ⓠ','R':'Ⓡ','S':'Ⓢ','T':'Ⓣ','U':'Ⓤ','V':'Ⓥ','W':'Ⓦ','X':'Ⓧ','Y':'Ⓨ','Z':'Ⓩ'}

const input = ref('Hello World')
const placeholder = '输入要转换的文本...'

const output = computed(() => {
  const map = type.value === 'upside' ? upsideMap : type.value === 'mirror' ? mirrorMap : type.value === 'bubble' ? bubbleMap : null
  if (map) {
    return input.value.split('').map((c) => map[c] || c).join('')
  } else if (type.value === 'strike') {
    return input.value.split('').map((c) => c + '\u0336').join('')
  } else if (type.value === 'wide') {
    return input.value.split('').map((c) => {
      const code = c.charCodeAt(0)
      if (code >= 33 && code <= 126) return String.fromCharCode(code + 0xFEE0)
      if (code === 32) return '\u3000'
      return c
    }).join('')
  }
  return input.value
})
</script>

<style scoped>
.flip { display: flex; flex-direction: column; gap: 12px; }
.flip__controls { display: flex; gap: 4px; flex-wrap: wrap; }
.flip__controls button { padding: 4px 12px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; }
.flip__controls button.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.flip__io { display: flex; gap: 12px; }
.flip__input { flex: 1; min-height: 100px; padding: 12px; border: 1px solid var(--border-color); border-radius: 8px; background: var(--bg-surface); color: var(--text-primary); font-size: 14px; resize: vertical; outline: none; }
.flip__output { flex: 1; padding: 12px; border: 1px solid var(--color-primary); border-radius: 8px; background: color-mix(in srgb, var(--color-primary) 8%, transparent); font-size: 16px; color: var(--color-primary); word-break: break-all; overflow-y: auto; }
</style>
