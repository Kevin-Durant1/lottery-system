<template>
  <div class="lottery-system">
    <h1>抽奖系统</h1>
    <div class="user-info">
      <p>点券余额：{{ dianquan }}</p>
      <p>幸运币：{{ luckbi }}</p>
      <p>已抽奖次数：{{ drawCount }}</p>
      <div class="statistics">
        <p>抽奖统计：</p>
        <p>传说：{{ legendaryCount }} 次 ({{ ((legendaryCount / drawCount) * 100).toFixed(2) || 0 }}%)</p>
        <p>史诗：{{ epicCount }} 次 ({{ ((epicCount / drawCount) * 100).toFixed(2) || 0 }}%)</p>
        <p>稀有：{{ rareCount }} 次 ({{ ((rareCount / drawCount) * 100).toFixed(2) || 0 }}%)</p>
        <p>普通：{{ normalCount }} 次 ({{ ((normalCount / drawCount) * 100).toFixed(2) || 0 }}%)</p>
      </div>
      <div class="buy-section">
        <input 
          type="number" 
          v-model="buyAmount" 
          min="1" 
          placeholder="输入购买数量"
        >
        <button 
          @click="buyMultipleLuckbi" 
          :disabled="dianquan < buyAmount * 10 || buyAmount <= 0"
        >
          购买幸运币 (需要{{ buyAmount * 10 }}点券)
        </button>
      </div>
    </div>
    
    <div class="lottery-section">
      <button @click="drawOnce" :disabled="luckbi <= 0">抽奖一次</button>
      <button @click="drawTen" :disabled="luckbi < 10">抽奖十次</button>
    </div>

    <div v-if="latestDrawResult" class="draw-result">
      <h3>抽奖结果：</h3>
      <p 
        v-for="(item, index) in latestDrawResult" 
        :key="index"
        :class="getItemClass(item)"
      >
        {{ item }}
      </p>
    </div>

    <div class="inventory">
      <h3>我的奖品库</h3>
      <ul>
        <li 
          v-for="(count, item) in groupedGoods" 
          :key="item"
          :class="getItemClass(item)"
        >
          {{ item }}{{ count > 1 ? ` *${count}` : '' }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'

const highGoods = ref('魔童闹海-哪吒')
const littleHighGoods = ref(['摩托车','scarl-魔童闹海哪吒','飞天猪飞行器','集结舱-玉虚云舱','哪吒降落伞'])
const commonGoods = ref(['头盔1级','头盔2级','头盔3级','背包1级','背包2级','背包3级','火尖枪','燃点复合弓','破片手雷弹','背包挂件'])
const lowGoods = ref(['物资币*20','量子晶体','纳米纤维','机械精密零件*10','枪械升级组件','枪械喷桶','璀璨碎片*20','璀璨碎片*6','璀璨碎片*3','璀璨碎片2','璀璨碎片*1'])
const dianquan = ref(99999)
const luckbi = ref(0)
// 修改 yourGoods 的类型定义
const yourGoods = ref<string[]>([]) // 明确指定为字符串数组类型
const latestDrawResult = ref<string[]>([])

// 购买幸运币
const buyLuckbi = () => {
  if (dianquan.value >= 10) {
    dianquan.value -= 10
    luckbi.value += 1
  }
}

// 批量购买幸运币
const buyMultipleLuckbi = () => {
  const cost = buyAmount.value * 10
  if (dianquan.value >= cost && buyAmount.value > 0) {
    dianquan.value -= cost
    luckbi.value += buyAmount.value
  }
}

// 抽奖函数
const draw = () => {
  const random = Math.random()
  if (random < 0.002) { // 0.2% 概率抽中最高级奖品
    return highGoods.value
  } else if (random < 0.0211) { // 1.91% 概率抽中高级奖品
    return littleHighGoods.value[Math.floor(Math.random() * littleHighGoods.value.length)]
  } else if (random < 0.2558) { // 23.47% 概率抽中普通奖品
    return commonGoods.value[Math.floor(Math.random() * commonGoods.value.length)]
  } else { // 74.6% 概率抽中低级奖品
    return lowGoods.value[Math.floor(Math.random() * lowGoods.value.length)]
  }
}

// 抽奖一次
// 抽奖统计
const drawCount = ref(0)
const legendaryCount = ref(0)
const epicCount = ref(0)
const rareCount = ref(0)
const normalCount = ref(0)

// 更新抽奖统计
const updateDrawStatistics = (prize: string) => {
  drawCount.value++
  if (prize === highGoods.value) {
    legendaryCount.value++
  } else if (littleHighGoods.value.includes(prize)) {
    epicCount.value++
  } else if (commonGoods.value.includes(prize)) {
    rareCount.value++
  } else {
    normalCount.value++
  }
}

// 修改抽奖函数
const drawOnce = () => {
  if (luckbi.value > 0) {
    luckbi.value--
    const prize = draw()
    yourGoods.value.push(prize)
    latestDrawResult.value = [prize]
    updateDrawStatistics(prize)
  }
}

const drawTen = () => {
  if (luckbi.value >= 10) {
    luckbi.value -= 10
    const results: string[] = []
    for (let i = 0; i < 10; i++) {
      const prize = draw()
      yourGoods.value.push(prize)
      results.push(prize)
      updateDrawStatistics(prize)
    }
    latestDrawResult.value = results
  }
}
const buyAmount = ref(1)

// 计算物品分组
const groupedGoods = computed(() => {
  const groups: Record<string, number> = {}
  yourGoods.value.forEach(item => {
    groups[item] = (groups[item] || 0) + 1
  })
  return groups
})

// 获取物品对应的样式类
const getItemClass = (item: string) => {
  if (item === highGoods.value) return 'legendary'
  if (littleHighGoods.value.includes(item)) return 'epic'
  if (commonGoods.value.includes(item)) return 'rare'
  return 'normal'
}
</script>

<style>
.lottery-system {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.user-info {
  margin: 20px 0;
}

.lottery-section {
  margin: 20px 0;
}

button {
  margin: 5px;
  padding: 8px 16px;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}

.draw-result {
  margin: 20px 0;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.inventory {
  margin-top: 30px;
}

.inventory ul {
  list-style: none;
  padding: 0;
}

.inventory li {
  padding: 5px 0;
  border-bottom: 1px solid #eee;
}

.buy-section {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 10px 0;
}

input[type="number"] {
  padding: 8px;
  width: 120px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* 物品稀有度样式 */
.legendary {
  color: #ff4d4f;
  font-weight: bold;
}

.epic {
  color: #722ed1;
  font-weight: bold;
}

.rare {
  color: #1890ff;
}

.normal {
  color: #52c41a;
}

.draw-result p {
  margin: 5px 0;
}

.statistics {
  margin: 15px 0;
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 5px;
}

.statistics p {
  margin: 5px 0;
}
</style>