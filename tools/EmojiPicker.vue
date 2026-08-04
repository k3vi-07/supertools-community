<template>
  <h-single-layout>
    <div class="emoji-picker">
      <h-input v-model="search" placeholder="搜索 Emoji... (如 smile, heart, cat)" />
      <div class="emoji-picker__categories">
        <button
          v-for="cat in categories"
          :key="cat.name"
          class="emoji-picker__cat-btn"
          :class="{ active: activeCategory === cat.name }"
          @click="activeCategory = cat.name"
        >{{ cat.icon }} {{ cat.label }}</button>
      </div>
      <div class="emoji-picker__grid">
        <button
          v-for="emoji in filteredEmojis"
          :key="emoji.char"
          class="emoji-picker__item"
          :title="emoji.name"
          @click="copy(emoji)"
        >{{ emoji.char }}</button>
      </div>
      <div v-if="filteredEmojis.length === 0" class="emoji-picker__empty">
        没有找到匹配的 Emoji
      </div>
    </div>
  </h-single-layout>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

interface Emoji { char: string; name: string; cat: string }
const search = ref('')
const activeCategory = ref('all')

const categories = [
  { name: 'all', label: '全部', icon: '📋' },
  { name: 'face', label: '表情', icon: '😀' },
  { name: 'gesture', label: '手势', icon: '👍' },
  { name: 'animal', label: '动物', icon: '🐱' },
  { name: 'food', label: '食物', icon: '🍎' },
  { name: 'activity', label: '活动', icon: '⚽' },
  { name: 'object', label: '物品', icon: '💡' },
  { name: 'symbol', label: '符号', icon: '❤️' }
]

const emojis: Emoji[] = [
  // 表情
  { char: '😀', name: 'smile happy 开心', cat: 'face' },
  { char: '😃', name: 'smile happy 开心', cat: 'face' },
  { char: '😄', name: 'happy 开心', cat: 'face' },
  { char: '😁', name: 'grin 笑', cat: 'face' },
  { char: '😆', name: 'laugh 笑', cat: 'face' },
  { char: '😅', name: 'sweat 尴尬', cat: 'face' },
  { char: '🤣', name: 'rofl 大笑', cat: 'face' },
  { char: '😂', name: 'joy 哭笑', cat: 'face' },
  { char: '🙂', name: 'smile 微笑', cat: 'face' },
  { char: '😉', name: 'wink 眨眼', cat: 'face' },
  { char: '😊', name: 'blush 害羞', cat: 'face' },
  { char: '😍', name: 'heart eyes 爱心', cat: 'face' },
  { char: '🥰', name: 'love 爱', cat: 'face' },
  { char: '😘', name: 'kiss 亲亲', cat: 'face' },
  { char: '😎', name: 'cool 酷', cat: 'face' },
  { char: '🤔', name: 'think 思考', cat: 'face' },
  { char: '🤨', name: 'suspicious 怀疑', cat: 'face' },
  { char: '😐', name: 'neutral 无语', cat: 'face' },
  { char: '😑', name: 'expressionless 面无表情', cat: 'face' },
  { char: '🙄', name: 'eye roll 翻白眼', cat: 'face' },
  { char: '😏', name: 'smirk 得意', cat: 'face' },
  { char: '😴', name: 'sleep 睡觉', cat: 'face' },
  { char: '😷', name: 'mask 生病', cat: 'face' },
  { char: '🤒', name: 'sick 发烧', cat: 'face' },
  { char: '🤓', name: 'nerd 书呆', cat: 'face' },
  { char: '😭', name: 'cry 哭', cat: 'face' },
  { char: '😢', name: 'sad 难过', cat: 'face' },
  { char: '😡', name: 'angry 生气', cat: 'face' },
  { char: '🤯', name: 'shocked 震惊', cat: 'face' },
  { char: '🥳', name: 'party 庆祝', cat: 'face' },
  // 手势
  { char: '👍', name: 'thumbs up 赞 好', cat: 'gesture' },
  { char: '👎', name: 'thumbs down 踩 差', cat: 'gesture' },
  { char: '👌', name: 'ok 好的', cat: 'gesture' },
  { char: '✌️', name: 'peace victory 胜利', cat: 'gesture' },
  { char: '🤞', name: 'fingers crossed 祈祷', cat: 'gesture' },
  { char: '🤟', name: 'love you 爱', cat: 'gesture' },
  { char: '🤙', name: 'call me 打电话', cat: 'gesture' },
  { char: '👋', name: 'wave 招手', cat: 'gesture' },
  { char: '🤚', name: 'raised hand 停', cat: 'gesture' },
  { char: '✋', name: 'stop 停止', cat: 'gesture' },
  { char: '👏', name: 'clap 鼓掌', cat: 'gesture' },
  { char: '🙌', name: 'raise hands 欢呼', cat: 'gesture' },
  { char: '🙏', name: 'pray 祈祷 感谢', cat: 'gesture' },
  { char: '💪', name: 'muscle 力量 加油', cat: 'gesture' },
  // 动物
  { char: '🐱', name: 'cat 猫', cat: 'animal' },
  { char: '🐶', name: 'dog 狗', cat: 'animal' },
  { char: '🐭', name: 'mouse 老鼠', cat: 'animal' },
  { char: '🐹', name: 'hamster 仓鼠', cat: 'animal' },
  { char: '🐰', name: 'rabbit 兔子', cat: 'animal' },
  { char: '🦊', name: 'fox 狐狸', cat: 'animal' },
  { char: '🐻', name: 'bear 熊', cat: 'animal' },
  { char: '🐼', name: 'panda 熊猫', cat: 'animal' },
  { char: '🐨', name: 'koala 考拉', cat: 'animal' },
  { char: '🐯', name: 'tiger 老虎', cat: 'animal' },
  { char: '🦁', name: 'lion 狮子', cat: 'animal' },
  { char: '🐮', name: 'cow 牛', cat: 'animal' },
  { char: '🐷', name: 'pig 猪', cat: 'animal' },
  { char: '🐸', name: 'frog 青蛙', cat: 'animal' },
  { char: '🐵', name: 'monkey 猴子', cat: 'animal' },
  { char: '🐔', name: 'chicken 鸡', cat: 'animal' },
  { char: '🐧', name: 'penguin 企鹅', cat: 'animal' },
  { char: '🐦', name: 'bird 鸟', cat: 'animal' },
  { char: '🦆', name: 'duck 鸭子', cat: 'animal' },
  { char: '🦉', name: 'owl 猫头鹰', cat: 'animal' },
  // 食物
  { char: '🍎', name: 'apple 苹果', cat: 'food' },
  { char: '🍊', name: 'orange 橘子', cat: 'food' },
  { char: '🍋', name: 'lemon 柠檬', cat: 'food' },
  { char: '🍌', name: 'banana 香蕉', cat: 'food' },
  { char: '🍉', name: 'watermelon 西瓜', cat: 'food' },
  { char: '🍇', name: 'grape 葡萄', cat: 'food' },
  { char: '🍓', name: 'strawberry 草莓', cat: 'food' },
  { char: '🍑', name: 'peach 桃子', cat: 'food' },
  { char: '🍒', name: 'cherry 樱桃', cat: 'food' },
  { char: '🥝', name: 'kiwi 猕猴桃', cat: 'food' },
  { char: '🍕', name: 'pizza 披萨', cat: 'food' },
  { char: '🍔', name: 'burger 汉堡', cat: 'food' },
  { char: '🍟', name: 'fries 薯条', cat: 'food' },
  { char: '🌭', name: 'hotdog 热狗', cat: 'food' },
  { char: '🍿', name: 'popcorn 爆米花', cat: 'food' },
  { char: '🍞', name: 'bread 面包', cat: 'food' },
  { char: '🥐', name: 'croissant 牛角包', cat: 'food' },
  { char: '☕', name: 'coffee 咖啡', cat: 'food' },
  { char: '🍺', name: 'beer 啤酒', cat: 'food' },
  { char: '🍷', name: 'wine 红酒', cat: 'food' },
  // 活动
  { char: '⚽', name: 'soccer 足球', cat: 'activity' },
  { char: '🏀', name: 'basketball 篮球', cat: 'activity' },
  { char: '🏈', name: 'football 橄榄球', cat: 'activity' },
  { char: '⚾', name: 'baseball 棒球', cat: 'activity' },
  { char: '🎾', name: 'tennis 网球', cat: 'activity' },
  { char: '🏐', name: 'volleyball 排球', cat: 'activity' },
  { char: '🎱', name: 'pool 台球', cat: 'activity' },
  { char: '🎮', name: 'game 游戏', cat: 'activity' },
  { char: '🎲', name: 'dice 骰子', cat: 'activity' },
  { char: '🎯', name: 'dart 飞镖', cat: 'activity' },
  { char: '🎵', name: 'music 音乐', cat: 'activity' },
  { char: '🎸', name: 'guitar 吉他', cat: 'activity' },
  { char: '🎹', name: 'piano 钢琴', cat: 'activity' },
  // 物品
  { char: '💡', name: 'bulb 灯泡 想法', cat: 'object' },
  { char: '📱', name: 'phone 手机', cat: 'object' },
  { char: '💻', name: 'laptop 笔记本', cat: 'object' },
  { char: '⌨️', name: 'keyboard 键盘', cat: 'object' },
  { char: '🖥️', name: 'desktop 电脑', cat: 'object' },
  { char: '🖱️', name: 'mouse 鼠标', cat: 'object' },
  { char: '🔋', name: 'battery 电池', cat: 'object' },
  { char: '📷', name: 'camera 相机', cat: 'object' },
  { char: '📺', name: 'tv 电视', cat: 'object' },
  { char: '🔔', name: 'bell 铃铛 通知', cat: 'object' },
  // 符号
  { char: '❤️', name: 'heart love 爱 心', cat: 'symbol' },
  { char: '🧡', name: 'orange heart', cat: 'symbol' },
  { char: '💛', name: 'yellow heart', cat: 'symbol' },
  { char: '💚', name: 'green heart', cat: 'symbol' },
  { char: '💙', name: 'blue heart', cat: 'symbol' },
  { char: '💜', name: 'purple heart', cat: 'symbol' },
  { char: '🖤', name: 'black heart', cat: 'symbol' },
  { char: '💔', name: 'broken heart 心碎', cat: 'symbol' },
  { char: '✅', name: 'check 正确', cat: 'symbol' },
  { char: '❌', name: 'cross 错误', cat: 'symbol' },
  { char: '⭐', name: 'star 星', cat: 'symbol' },
  { char: '🌟', name: 'glowing star', cat: 'symbol' },
  { char: '⚡', name: 'zap lightning 闪电', cat: 'symbol' },
  { char: '🔥', name: 'fire 火 热门', cat: 'symbol' },
  { char: '💯', name: 'hundred 满分', cat: 'symbol' },
  { char: '✨', name: 'sparkles 闪光', cat: 'symbol' },
  { char: '🎉', name: 'party 庆祝 派对', cat: 'symbol' },
  { char: '🎁', name: 'gift 礼物', cat: 'symbol' },
  { char: '🚀', name: 'rocket 火箭', cat: 'symbol' },
  { char: '🌈', name: 'rainbow 彩虹', cat: 'symbol' }
]

const filteredEmojis = computed(() => {
  let list = emojis
  if (activeCategory.value !== 'all') {
    list = list.filter((e) => e.cat === activeCategory.value)
  }
  if (search.value.trim()) {
    const q = search.value.toLowerCase()
    list = list.filter((e) => e.name.includes(q))
  }
  return list
})

function copy(emoji: Emoji): void {
  window.$he3?.copyText(emoji.char)
  window.$he3?.message.success(`已复制 ${emoji.char}`)
}
</script>

<style scoped>
.emoji-picker { display: flex; flex-direction: column; gap: 12px; }
.emoji-picker__categories { display: flex; gap: 4px; flex-wrap: wrap; }
.emoji-picker__cat-btn { padding: 4px 10px; border: 1px solid var(--border-color); border-radius: 6px; background: var(--bg-surface); color: var(--text-secondary); font-size: 12px; cursor: pointer; transition: all 0.15s; }
.emoji-picker__cat-btn:hover { border-color: var(--color-primary); }
.emoji-picker__cat-btn.active { background: var(--color-primary); color: white; border-color: var(--color-primary); }
.emoji-picker__grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(40px, 1fr)); gap: 4px; max-height: 400px; overflow-y: auto; }
.emoji-picker__item { display: flex; align-items: center; justify-content: center; width: 40px; height: 40px; border: none; border-radius: 6px; background: var(--bg-surface); font-size: 22px; cursor: pointer; transition: all 0.15s; }
.emoji-picker__item:hover { background: var(--bg-hover); transform: scale(1.2); }
.emoji-picker__empty { padding: 40px; text-align: center; color: var(--text-tertiary); font-size: 14px; }
</style>
