<script setup>
import { ref, computed, watch, watchEffect } from 'vue'

// 1. 날씨 데이터 배열
const weatherList = ref([
  { id: 'city_01', name: '서울', temp: 28, status: '맑음' },
  { id: 'city_02', name: '수원', temp: 24, status: '비' },
  { id: 'city_03', name: '부산', temp: 26, status: '구름' },
])

// 2. 상태 관리 변수들
const searchQuery = ref('')
const selectedCityInfo = ref('카드를 클릭하거나 검색해 보세요.')

// 3. computed를 활용한 검색 도시 필터링
const filteredWeatherList = computed(() => {
  if (!searchQuery.value) {
    return weatherList.value
  }
  return weatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

// 4-1. watch: selectedCityInfo의 변경을 감시하여 콘솔 로그 출력
watch(selectedCityInfo, (newValue, oldValue) => {
  console.log(`[watch 감지] 상태 바 문구가 업데이트되었습니다 -> "${newValue}"`)
})

// 4-2. watchEffect: searchQuery를 타이핑할 때마다 자동으로 감지하여 콘솔 로그 출력
watchEffect(() => {
  console.log(
    `[watchEffect 자동 호출] 현재 검색어 "${searchQuery.value}" 에 매칭되는 API 데이터를 탐색합니다...`,
  )
})

// 날씨 상태에 따른 이모지 아이콘 반환 함수
const getWeatherIcon = (status) => {
  if (status === '맑음') return '☀️'
  if (status === '비') return '🌧️'
  if (status === '구름') return '☁️'
  return '🌡️'
}

// 카드 클릭 시 상태 바 업데이트 함수
const selectCity = (cityName) => {
  selectedCityInfo.value = `${cityName}이 선택되었습니다.`
}

// 상세보기 버튼 클릭 시 실행될 함수
const showDetail = (cityName, status) => {
  window.alert(`${cityName}의 현재 날씨는 [${status}] 상태입니다.`)
}
</script>

<template>
  <div style="padding: 20px; max-width: 600px; margin: 0 auto; color: #333">
    <h2 style="color: #fff">과제: 날씨🌟</h2>

    <!-- 도시 검색 입력창 -->
    <div
      style="
        margin-bottom: 20px;
        background: #ffffff;
        padding: 15px;
        border-radius: 8px;
        color: #333;
      "
    >
      <label style="font-weight: bold; color: #333">도시 검색</label><br />
      <input
        type="text"
        v-model="searchQuery"
        placeholder="검색할 도시 이름 입력 (예: 수원)"
        style="
          padding: 8px;
          width: 100%;
          margin-top: 5px;
          box-sizing: border-box;
          background: #fff;
          color: #333;
          border: 1px solid #ccc;
        "
      />
      <p style="margin-top: 5px; font-size: 14px; color: #555">
        검색 중인 도시: <strong>{{ searchQuery || '전체 조회 중' }}</strong>
      </p>
    </div>

    <!-- 지역별 날씨 현황 목록 -->
    <h3 style="color: #fff">지역별 날씨 현황</h3>

    <!-- 검색 결과가 없을 때 안내 문구 -->
    <p
      v-if="filteredWeatherList.length === 0"
      style="color: #ff6b6b; text-align: center; padding: 20px; font-weight: bold"
    >
      🔍 검색 결과가 일치하는 도시가 없습니다.
    </p>

    <!-- v-for를 이용한 배열 렌더링 -->
    <div
      v-for="city in filteredWeatherList"
      :key="city.id"
      @click="selectCity(city.name)"
      style="
        background: #ffffff;
        color: #222222;
        border: 1px solid #ddd;
        padding: 15px;
        margin-bottom: 10px;
        border-radius: 8px;
        cursor: pointer;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      "
    >
      <h4 style="margin: 0 0 8px 0; color: #111; font-size: 18px">
        {{ getWeatherIcon(city.status) }} {{ city.name }} ({{ city.status }})
      </h4>
      <p style="margin: 0 0 10px 0; color: #444">현재 기온: {{ city.temp }}°C</p>

      <!-- 조건부 렌더링 라벨 -->
      <span
        v-if="city.temp >= 25"
        style="
          background: #ffebee;
          color: #c62828;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 12px;
          font-weight: bold;
        "
      >
        🔥 더움 (25도 이상)
      </span>
      <span
        v-else
        style="
          background: #e3f2fd;
          color: #1565c0;
          padding: 4px 8px;
          border-radius: 4px;
          font-size: 12px;
          font-weight: bold;
        "
      >
        ❄️ 선선함 (25도 미만)
      </span>

      <!-- 상세보기 버튼 -->
      <button
        @click.stop="showDetail(city.name, city.status)"
        style="
          float: right;
          padding: 6px 12px;
          cursor: pointer;
          background: #f0f0f0;
          border: 1px solid #ccc;
          border-radius: 4px;
          font-weight: bold;
          color: #333;
        "
      >
        상세보기
      </button>
      <div style="clear: both"></div>
    </div>

    <!-- 하단 상태 바 (watch 감지 대상) -->
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
