<template>
  <div class="min-h-screen bg-gray-900 text-gray-100 p-4 md:p-8 font-sans">
    <header class="mb-8 border-l-4 border-red-600 pl-4">
      <h1 class="text-3xl font-bold tracking-tight">T1 四月主場日購物紀錄</h1>
      <p class="text-gray-400">專為電腦與手機優化的記帳系統</p>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-8">
      <div class="bg-gray-800 p-6 rounded-2xl border border-gray-700">
        <p class="text-sm text-gray-400">目前條件下總金額</p>
        <p class="text-4xl font-black text-white">NT$ {{ filteredTotal }}</p>
      </div>
      <div class="bg-gray-800 p-6 rounded-2xl border border-gray-700">
        <p class="text-sm text-gray-400">剩餘待付金額</p>
        <p class="text-4xl font-black text-red-500">NT$ {{ pendingTotal }}</p>
      </div>
    </div>

    <div class="bg-gray-800 p-4 rounded-xl mb-8 grid grid-cols-1 sm:grid-cols-3 gap-4">
      <div>
        <label class="block text-xs text-gray-500 mb-1">篩選購買日</label>
        <select v-model="filter.date" class="w-full bg-gray-700 rounded-lg p-2 border-none">
          <option value="">全部日期</option>
          <option v-for="d in ['4/24', '4/25', '4/26']" :key="d">{{ d }}</option>
        </select>
      </div>
      <div>
        <label class="block text-xs text-gray-500 mb-1">篩選購買人</label>
        <select v-model="filter.purchaser" class="w-full bg-gray-700 rounded-lg p-2 border-none">
          <option value="">全部人員</option>
          <option v-for="p in ['Nio', '慕斯', '阿龍']" :key="p">{{ p }}</option>
        </select>
      </div>
      <div>
        <label class="block text-xs text-gray-500 mb-1">篩選選手</label>
        <select v-model="filter.player" class="w-full bg-gray-700 rounded-lg p-2 border-none">
          <option value="">全部選手</option>
          <option v-for="p in ['無', 'Doran', 'Oner', 'Faker', 'Peyz', 'Keria']" :key="p">{{ p }}</option>
        </select>
      </div>
    </div>

    <div class="overflow-x-auto bg-gray-800 rounded-2xl border border-gray-700">
      <table class="w-full text-left border-collapse">
        <thead class="bg-gray-700">
          <tr>
            <th class="p-4">內容</th>
            <th class="p-4">價格 (TWD)</th>
            <th class="p-4 text-center">已付</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in filteredRecords" :key="item.ID" class="border-t border-gray-700 hover:bg-gray-750 transition-colors">
            <td class="p-4">
              <div class="font-bold">{{ item.商品名稱 }}</div>
              <div class="text-xs text-gray-400">{{ item.購買日 }} · {{ item.購買人 }} · {{ item.選手 }}</div>
            </td>
            <td class="p-4 font-mono">
              ${{ item.台幣價格 }}
            </td>
            <td class="p-4 text-center text-xl">
              {{ item.是否付費 ? '✅' : '⏳' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const GAS_URL = '你的_GAS_WEB_APP_URL'
const records = ref([])
const filter = ref({
  date: '',
  purchaser: '',
  player: ''
})

// 篩選邏輯
const filteredRecords = computed(() => {
  return records.value.filter(r => {
    return (!filter.value.date || r.購買日 === filter.value.date) &&
           (!filter.value.purchaser || r.購買人 === filter.value.purchaser) &&
           (!filter.value.player || r.選手 === filter.value.player)
  })
})

// 自動加總
const filteredTotal = computed(() => {
  return filteredRecords.value.reduce((sum, r) => sum + Number(r.台幣價格), 0)
})

// 待付金額 (扣除已勾選已付費的)
const pendingTotal = computed(() => {
  return filteredRecords.value
    .filter(r => !r.是否付費)
    .reduce((sum, r) => sum + Number(r.台幣價格), 0)
})

// 抓取資料
const fetchRecords = async () => {
  const res = await fetch(GAS_URL)
  records.value = await res.json()
}

onMounted(fetchRecords)
</script>

<style>
/* 深色背景優化 */
body { background-color: #111827; }
</style>
