<script setup>
import { ref, computed, watch, watchEffect } from 'vue'
import BaseDashboardCard from '@/components/BaseDashboardCard.vue'
import SearchBar from '@/components/SearchBar.vue'
import WeatheCard from '@/components/WeatheCard.vue'

// 반응형 데이터
const weatherList = ref([
  { id: 'city_01', name: '서울', temp: 28, status: '맑음' },
  { id: 'city_02', name: '수원', temp: 24, status: '비' },
  { id: 'city_03', name: '부산', temp: 26, status: '구름' },
])

const searchQuery = ref('')
const selectedCityInfo = ref('카드를 클릭하거나 검색해 보세요.')

// Computed 필터링
const filteredWeatherList = computed(() => {
  if (!searchQuery.value) {
    return weatherList.value
  }
  return weatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

// Watch & WatchEffect 감시자
watch(selectedCityInfo, (newValue) => {
  console.log(`[watch 감지] 상태 바 문구가 업데이트되었습니다 -> "${newValue}"`)
})

watchEffect(() => {
  console.log(
    `[watchEffect 자동 호출] 현재 검색어 "${searchQuery.value}" 에 매칭되는 데이터를 탐색합니다...`,
  )
})

// 이벤트 핸들러
const handleSelectCard = (cityName) => {
  selectedCityInfo.value = `${cityName}이 선택되었습니다.`
}

const handleDetail = (cityName, status) => {
  window.alert(`${cityName}의 현재 날씨는 [${status}] 상태입니다.`)
}
</script>

<template>
  <div style="padding: 20px; max-width: 600px; margin: 0 auto; color: #333">
    <h2 style="color: #fff">과제 3: 날씨 (컴포넌트 분리) 🌟</h2>

    <!-- 1. 검색바 컴포넌트 호출 (v-model 양방향 바인딩) -->
    <SearchBar v-model="searchQuery" />

    <!-- ✨ 수정된 제목 영역 -->
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

    <!-- 2. 공통 카드 틀(BaseDashboardCard) 안에 날씨 카드(WeatheCard) 쏙 넣기 (Slot 활용) -->
    <BaseDashboardCard v-for="city in filteredWeatherList" :key="city.id">
      <WeatheCard :city="city" @select-card="handleSelectCard" @click-detail="handleDetail" />
    </BaseDashboardCard>

    <!-- 하단 상태 바 -->
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

<style scoped></style>
