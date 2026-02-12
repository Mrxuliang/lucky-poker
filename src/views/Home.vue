<template>
  <div class="home-container">
    <div class="header">
      <h1 class="title">🧧 春节打牌运势 🧧</h1>
      <p class="subtitle">{{ currentDate }} {{ lunarDate }}</p>
    </div>

    <div class="content">
      <div class="orientation-section" v-if="orientationStatus">
        <div class="compass-display">
          <div class="compass-ring">
            <div class="compass-needle" :style="{ transform: `rotate(${heading}deg)` }"></div>
            <span class="direction-text">{{ direction }}</span>
          </div>
        </div>
        <p class="orientation-tip">{{ orientationTip }}</p>
      </div>

      <div class="poker-grid">
        <div 
          v-for="(poker, index) in pokerTypes" 
          :key="index"
          class="poker-card"
          @click="showPokerFortune(poker)"
        >
          <div class="poker-icon">{{ poker.icon }}</div>
          <h3 class="poker-name">{{ poker.name }}</h3>
          <div class="lucky-score">
            <span class="score-label">今日幸运指数</span>
            <div class="score-bar">
              <div class="score-fill" :style="{ width: getLuckyScore(poker.id) + '%' }"></div>
            </div>
            <span class="score-value">{{ getLuckyScore(poker.id) }}</span>
          </div>
        </div>
      </div>

      <div class="history-section">
        <h3 class="section-title">📜 历史运势</h3>
        <div class="history-list" v-if="historyList.length > 0">
          <div v-for="(item, index) in historyList.slice(0, 5)" :key="index" class="history-item">
            <span class="history-date">{{ item.date }}</span>
            <span class="history-poker">{{ item.poker }}</span>
            <span class="history-score" :class="getScoreClass(item.score)">{{ item.score }}分</span>
          </div>
        </div>
        <p class="empty-history" v-else>暂无历史记录</p>
      </div>
    </div>

    <div class="modal" v-if="showModal" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h2>{{ selectedPoker?.icon }} {{ selectedPoker?.name }}运势</h2>
          <button class="close-btn" @click="closeModal">×</button>
        </div>
        <div class="modal-body" v-if="selectedPoker">
          <div class="fortune-score">
            <div class="score-circle">
              <span class="score-number">{{ getLuckyScore(selectedPoker.id) }}</span>
              <span class="score-label">分</span>
            </div>
            <p class="fortune-level">{{ getFortuneLevel(getLuckyScore(selectedPoker.id)) }}</p>
          </div>
          
          <div class="fortune-details">
            <div class="detail-item">
              <span class="detail-label">🧭 建议方位</span>
              <span class="detail-value">{{ getBestDirection() }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">⏰ 最佳时间</span>
              <span class="detail-value">{{ getBestTime() }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">🎯 幸运数字</span>
              <span class="detail-value">{{ getLuckyNumber() }}</span>
            </div>
            <div class="detail-item">
              <span class="detail-label">💰 开运方法</span>
              <span class="detail-value">{{ getTip(selectedPoker.id) }}</span>
            </div>
          </div>

          <div class="fortune-advice">
            <h4>今日运势解析</h4>
            <p>{{ getFortuneAdvice(selectedPoker.id) }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const currentDate = ref('')
const lunarDate = ref('')
const heading = ref(0)
const direction = ref('')
const orientationStatus = ref(false)
const showModal = ref(false)
const selectedPoker = ref(null)
const historyList = ref([])

const pokerTypes = [
  { id: 'texas', name: '德州扑克', icon: '🃏' },
  { id: 'mahjong', name: '麻将', icon: '🀄' },
  { id: 'douzhu', name: '斗地主', icon: '👑' },
  { id: 'zajinhua', name: '炸金花', icon: '💥' }
]

const orientationTip = computed(() => {
  if (!orientationStatus.value) return '点击下方卡片查看详细运势'
  return `当前方位: ${direction.value} | 建议面向: ${getBestDirection()}`
})

onMounted(() => {
  updateDate()
  loadHistory()
  initOrientation()
})

function updateDate() {
  const now = new Date()
  currentDate.value = `${now.getFullYear()}年${now.getMonth() + 1}月${now.getDate()}日`
  lunarDate.value = getLunarDate(now)
}

function getLunarDate(date) {
  const lunarDays = ['初一', '初二', '初三', '初四', '初五', '初六', '初七', '初八', '初九', '初十',
                    '十一', '十二', '十三', '十四', '十五', '十六', '十七', '十八', '十九', '二十',
                    '廿一', '廿二', '廿三', '廿四', '廿五', '廿六', '廿七', '廿八', '廿九', '三十']
  const lunarMonths = ['正月', '二月', '三月', '四月', '五月', '六月', '七月', '八月', '九月', '十月', '冬月', '腊月']
  
  const baseDate = new Date(2024, 1, 10)
  const diffTime = date.getTime() - baseDate.getTime()
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24))
  
  const lunarDayIndex = (diffDays % 30 + 30) % 30
  const lunarMonthIndex = Math.floor((diffDays % 365) / 29.5)
  
  return `${lunarMonths[lunarMonthIndex]}${lunarDays[lunarDayIndex]}`
}

function initOrientation() {
  if (typeof DeviceOrientationEvent !== 'undefined' && 
      typeof DeviceOrientationEvent.requestPermission === 'function') {
    DeviceOrientationEvent.requestPermission()
      .then(response => {
        if (response === 'granted') {
          window.addEventListener('deviceorientation', handleOrientation)
          orientationStatus.value = true
        }
      })
      .catch(console.error)
  } else if ('ondeviceorientationabsolute' in window) {
    window.addEventListener('deviceorientationabsolute', handleOrientation)
    orientationStatus.value = true
  } else if ('ondeviceorientation' in window) {
    window.addEventListener('deviceorientation', handleOrientation)
    orientationStatus.value = true
  }
}

function handleOrientation(event) {
  if (event.alpha !== null) {
    heading.value = event.alpha
    direction.value = getDirectionText(event.alpha)
  }
}

function getDirectionText(angle) {
  const directions = ['北', '东北', '东', '东南', '南', '西南', '西', '西北']
  const index = Math.round(((angle % 360) + 360) % 360 / 45) % 8
  return directions[index]
}

function getLuckyScore(pokerId) {
  const now = new Date()
  const daySum = now.getFullYear() + now.getMonth() + now.getDate()
  const pokerIndex = pokerTypes.findIndex(p => p.id === pokerId)
  const baseScore = ((daySum * (pokerIndex + 1)) % 41) + 60
  return Math.min(99, baseScore)
}

function getFortuneLevel(score) {
  if (score >= 90) return '🌟 大吉大利'
  if (score >= 80) return '✨ 运气爆棚'
  if (score >= 70) return '🔥 吉星高照'
  if (score >= 60) return '🍀 小有收获'
  return '⚡ 需谨慎'
}

function getBestDirection() {
  const now = new Date()
  const directions = ['正东', '正南', '正西', '正北', '东南', '东北', '西南', '西北']
  return directions[now.getDate() % 8]
}

function getBestTime() {
  const now = new Date()
  const times = ['9:00-11:00', '14:00-16:00', '19:00-21:00', '21:00-23:00']
  return times[now.getDate() % 4]
}

function getLuckyNumber() {
  const now = new Date()
  const num1 = (now.getDate() % 9) + 1
  const num2 = ((now.getDate() * 2) % 9) + 1
  return `${num1}, ${num2}`
}

function getTip(pokerId) {
  const tips = {
    texas: '保持冷静，观察对手，适时加注',
    mahjong: '多听大牌，注意听牌时机',
    douzhu: '记住出过的牌，合理使用炸弹',
    zajinhua: '虚张声势，看准时机下注'
  }
  return tips[pokerId]
}

function getFortuneAdvice(pokerId) {
  const score = getLuckyScore(pokerId)
  const now = new Date()
  
  if (score >= 80) {
    return `今日${pokerTypes.find(p => p.id === pokerId).name}运势极佳！适合积极参与，把握机会。${getBestDirection()}方位运气最佳，${getBestTime()}时段运势最强。`
  } else if (score >= 60) {
    return `今日${pokerTypes.find(p => p.id === pokerId).name}运势平稳。建议保持谨慎，${getBestDirection()}方位可能带来好运。`
  } else {
    return `今日${pokerTypes.find(p => p.id === pokerId).name}运势一般，建议适度参与，避免大额下注。建议多观察，少出手。`
  }
}

function showPokerFortune(poker) {
  selectedPoker.value = poker
  showModal.value = true
  
  const historyItem = {
    date: currentDate.value,
    poker: poker.name,
    score: getLuckyScore(poker.id)
  }
  
  const existingIndex = historyList.value.findIndex(
    h => h.date === historyItem.date && h.poker === historyItem.poker
  )
  
  if (existingIndex === -1) {
    historyList.value.unshift(historyItem)
    if (historyList.value.length > 30) {
      historyList.value = historyList.value.slice(0, 30)
    }
    saveHistory()
  }
}

function closeModal() {
  showModal.value = false
  selectedPoker.value = null
}

function getScoreClass(score) {
  if (score >= 80) return 'high'
  if (score >= 60) return 'medium'
  return 'low'
}

function saveHistory() {
  localStorage.setItem('pokerHistory', JSON.stringify(historyList.value))
}

function loadHistory() {
  const saved = localStorage.getItem('pokerHistory')
  if (saved) {
    historyList.value = JSON.parse(saved)
  }
}
</script>

<style scoped>
.home-container {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(180deg, #C41E3A 0%, #8B0000 100%);
  padding: 20px;
  padding-bottom: 40px;
}

.header {
  text-align: center;
  padding: 30px 0;
}

.title {
  font-size: 28px;
  color: #FFD700;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.subtitle {
  font-size: 16px;
  color: #FFF8DC;
}

.content {
  max-width: 600px;
  margin: 0 auto;
}

.orientation-section {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 30px;
  backdrop-filter: blur(10px);
}

.compass-display {
  display: flex;
  justify-content: center;
  margin-bottom: 15px;
}

.compass-ring {
  width: 120px;
  height: 120px;
  border: 3px solid #FFD700;
  border-radius: 50%;
  position: relative;
  background: rgba(0, 0, 0, 0.2);
}

.compass-needle {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 4px;
  height: 45px;
  background: linear-gradient(to bottom, #FFD700 50%, #8B0000 50%);
  transform-origin: bottom center;
  transform: translateX(-50%) translateY(-100%);
  border-radius: 2px;
}

.direction-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #FFD700;
  font-size: 18px;
  font-weight: bold;
}

.orientation-tip {
  text-align: center;
  color: #FFF8DC;
  font-size: 14px;
}

.poker-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 30px;
}

.poker-card {
  background: linear-gradient(135deg, #FFD700 0%, #FFA500 100%);
  border-radius: 15px;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  transition: transform 0.3s, box-shadow 0.3s;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.poker-card:active {
  transform: scale(0.95);
}

.poker-icon {
  font-size: 48px;
  margin-bottom: 10px;
}

.poker-name {
  font-size: 18px;
  color: #8B0000;
  margin-bottom: 15px;
  font-weight: bold;
}

.lucky-score {
  background: rgba(139, 0, 0, 0.2);
  border-radius: 10px;
  padding: 10px;
}

.score-label {
  display: block;
  font-size: 12px;
  color: #8B0000;
  margin-bottom: 5px;
}

.score-bar {
  height: 8px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 5px;
}

.score-fill {
  height: 100%;
  background: linear-gradient(90deg, #FF6B6B, #4ECDC4);
  transition: width 0.5s ease;
}

.score-value {
  font-size: 20px;
  font-weight: bold;
  color: #8B0000;
}

.history-section {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 20px;
  backdrop-filter: blur(10px);
}

.section-title {
  font-size: 18px;
  color: #FFD700;
  margin-bottom: 15px;
}

.history-list {
  max-height: 200px;
  overflow-y: auto;
}

.history-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid rgba(255, 215, 0, 0.2);
}

.history-item:last-child {
  border-bottom: none;
}

.history-date {
  color: #FFF8DC;
  font-size: 14px;
  flex: 1;
}

.history-poker {
  color: #FFD700;
  font-size: 14px;
  flex: 1;
  text-align: center;
}

.history-score {
  font-size: 16px;
  font-weight: bold;
  flex: 0 0 50px;
  text-align: right;
}

.history-score.high {
  color: #4ECDC4;
}

.history-score.medium {
  color: #FFD700;
}

.history-score.low {
  color: #FF6B6B;
}

.empty-history {
  text-align: center;
  color: #FFF8DC;
  padding: 20px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.modal-content {
  background: linear-gradient(135deg, #C41E3A 0%, #8B0000 100%);
  border-radius: 20px;
  max-width: 400px;
  width: 100%;
  max-height: 90vh;
  overflow-y: auto;
  border: 2px solid #FFD700;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid rgba(255, 215, 0, 0.3);
}

.modal-header h2 {
  color: #FFD700;
  font-size: 20px;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: #FFD700;
  font-size: 30px;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
  line-height: 30px;
}

.modal-body {
  padding: 20px;
}

.fortune-score {
  text-align: center;
  margin-bottom: 25px;
}

.score-circle {
  width: 120px;
  height: 120px;
  border: 4px solid #FFD700;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 0 auto 15px;
  background: rgba(0, 0, 0, 0.2);
}

.score-number {
  font-size: 40px;
  font-weight: bold;
  color: #FFD700;
}

.score-label {
  font-size: 14px;
  color: #FFF8DC;
}

.fortune-level {
  font-size: 18px;
  color: #FFD700;
  font-weight: bold;
}

.fortune-details {
  margin-bottom: 25px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid rgba(255, 215, 0, 0.2);
}

.detail-label {
  color: #FFF8DC;
  font-size: 14px;
}

.detail-value {
  color: #FFD700;
  font-size: 14px;
  font-weight: bold;
}

.fortune-advice {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  padding: 15px;
}

.fortune-advice h4 {
  color: #FFD700;
  margin-bottom: 10px;
  font-size: 16px;
}

.fortune-advice p {
  color: #FFF8DC;
  font-size: 14px;
  line-height: 1.6;
}
</style>