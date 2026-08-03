<script setup>
import { ref, computed, watch, watchEffect } from 'vue'
import { useRouter } from 'vue-router'
import BaseDashboardCard from '@/components/exercise/BaseDashboardCard.vue'
import SearchBar from '@/components/exercise/SearchBar.vue'
import WeatheCard from '@/components/exercise/WeatheCard.vue'

const router = useRouter()

const weatherList = ref([
  { id: 'city_01', name: '서울', temp: 28, status: '맑음' },
  { id: 'city_02', name: '수원', temp: 24, status: '비' },
  { id: 'city_03', name: 'West-lafayette', temp: 10, status: '맑음' },
  { id: 'city_04', name: '부산', temp: 26, status: '구름' },
  { id: 'city_05', name: '대구', temp: 31, status: '맑음' },
  { id: 'city_06', name: '제주', temp: 25, status: '구름' },
])

const searchQuery = ref('')
const selectedCityInfo = ref('카드를 클릭하거나 검색해 보세요.')

const filteredWeatherList = computed(() => {
  if (!searchQuery.value) return weatherList.value
  return weatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

watch(selectedCityInfo, (newValue) => {
  console.log(`[watch 감지] 상태 바 문구가 업데이트되었습니다 -> "${newValue}"`)
})

watchEffect(() => {
  console.log(
    `[watchEffect 자동 호출] 현재 검색어 "${searchQuery.value}" 에 매칭되는 데이터를 탐색합니다...`,
  )
})

const handleSelectCard = (cityName) => {
  selectedCityInfo.value = `${cityName}이 선택되었습니다.`
}

// 🌟 상세보기 클릭 시 alert 대신 동적 라우터 경로로 페이지 이동!
const handleDetail = (cityId, cityName, status) => {
  router.push(`/weather/${cityId}`)
}

const getCardTheme = (status, temp) => {
  if (status === '비') return { bg: '#f0f7ff', border: '#bae6fd' }
  if (status === '구름') return { bg: '#f8fafc', border: '#cbd5e1' }
  if (temp >= 28) return { bg: '#fff5f5', border: '#feb2b2' }
  return { bg: '#fffbeb', border: '#fde68a' }
}
</script>

<template>
  <div style="padding: 20px; max-width: 600px; margin: 0 auto; color: #333">
    <SearchBar v-model="searchQuery" />

    <h3 style="color: #fff; margin-top: 20px; margin-bottom: 10px">
      지역별 날씨 현황
      <span
        style="
          display: inline-block;
          font-size: 13px;
          background: #4caf50;
          color: #ffffff;
          padding: 2px 10px;
          border-radius: 12px;
          margin-left: 8px;
          font-weight: normal;
        "
      >
        {{ filteredWeatherList.length }}개 도시
      </span>
    </h3>

    <p
      v-if="filteredWeatherList.length === 0"
      style="color: #ff6b6b; text-align: center; padding: 20px; font-weight: bold"
    >
      🔍 검색 결과가 일치하는 도시가 없습니다.
    </p>

    <BaseDashboardCard
      v-for="city in filteredWeatherList"
      :key="city.id"
      :bgColor="getCardTheme(city.status, city.temp).bg"
      :borderColor="getCardTheme(city.status, city.temp).border"
    >
      <WeatheCard
        :city="city"
        @select-card="handleSelectCard"
        @click-detail="(cityName, status) => handleDetail(city.id, cityName, status)"
      />
    </BaseDashboardCard>

    <div
      style="
        margin-top: 20px;
        background: #e8f5e9;
        color: #2e7d32;
        padding: 12px;
        border-radius: 8px;
        font-weight: bold;
        text-align: center;
        border: 1px solid #c8e6c9;
      "
    >
      {{ selectedCityInfo }}
    </div>
  </div>
</template>
