<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import axios from 'axios'

const route = useRoute()
const router = useRouter()

// 상태 변수 정의
const weatherInfo = ref(null)
const isLoading = ref(false)
const errorMessage = ref('')

// 🌟 URL로 넘어오는 모든 형태의 값(city_03, West-lafayette 등)을 커버하도록 매핑 보완
const cityCoordinates = {
  city_01: { name: '서울특별시', lat: 37.5665, lon: 126.978 },
  서울: { name: '서울특별시', lat: 37.5665, lon: 126.978 },

  city_03: { name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },
  'West-lafayette': { name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },
  'West Lafayette': { name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },

  city_04: { name: '부산광역시', lat: 35.1796, lon: 129.0756 },
  부산: { name: '부산광역시', lat: 35.1796, lon: 129.0756 },

  city_05: { name: '대구광역시', lat: 35.8714, lon: 128.6014 },
  대구: { name: '대구광역시', lat: 35.8714, lon: 128.6014 },
}

const currentCityMeta = cityCoordinates[route.params.cityId] || {
  name: route.params.cityId || '알 수 없는 도시',
  lat: 37.5665,
  lon: 126.978,
}

// 🌟 핵심: 화면이 마운트될 때 OpenWeather API 호출!
const fetchRealWeather = async () => {
  isLoading.value = true
  const API_KEY = '31df255e3ec6209feb7ed06f52a7a1f8'
  const URL = `https://api.openweathermap.org/data/2.5/weather?lat=${currentCityMeta.lat}&lon=${currentCityMeta.lon}&appid=${API_KEY}&units=metric&lang=kr`

  try {
    const response = await axios.get(URL)
    console.log('Axios 통신 성공 응답 전체:', response)
    weatherInfo.value = response.data
  } catch (error) {
    console.error('통신 중 에러가 발생했습니다:', error)
    errorMessage.value = '데이터를 가져오지 못했습니다. API 키나 네트워크를 확인하세요.'
  } finally {
    isLoading.value = false
  }
}

onMounted(() => {
  fetchRealWeather()
})
</script>

<template>
  <div
    style="
      padding: 30px 20px;
      max-width: 600px;
      margin: 0 auto;
      background: #ffffff;
      border-radius: 12px;
      color: #1f2937;
      border: 1px solid #e5e7eb;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
    "
  >
    <h2>🌍 실시간 OpenWeather API 관측 정보</h2>

    <!-- 로딩 중일 때 -->
    <p
      v-if="isLoading"
      style="text-align: center; padding: 30px; color: #2563eb; font-weight: bold; font-size: 16px"
    >
      🔄 실시간 날씨 데이터를 불러오는 중입니다...
    </p>

    <!-- 에러 발생 시 -->
    <p
      v-else-if="errorMessage"
      style="color: #dc2626; font-weight: bold; text-align: center; padding: 30px; font-size: 16px"
    >
      {{ errorMessage }}
    </p>

    <!-- 데이터 로드 성공 시 -->
    <div
      v-else-if="weatherInfo"
      style="
        margin-top: 20px;
        background: #f8fafc;
        padding: 25px;
        border-radius: 8px;
        border: 1px solid #cbd5e1;
      "
    >
      <p style="font-size: 20px; font-weight: bold; margin-bottom: 20px; color: #111827">
        📍 지점 지역: {{ currentCityMeta.name }}
      </p>
      <p style="font-size: 16px; line-height: 1.6">
        🌡️ <strong>현재 기온:</strong>
        <span style="font-size: 22px; font-weight: bold; color: #b91c1c"
          >{{ weatherInfo.main.temp }}°C</span
        >
        (체감 온도: {{ weatherInfo.main.feels_like }}°C)
      </p>
      <p style="font-size: 16px; line-height: 1.6">
        ☁️ <strong>기상 현황:</strong> {{ weatherInfo.weather[0].description }}
      </p>
      <p style="font-size: 16px; line-height: 1.6">
        💧 <strong>대기 습도:</strong> {{ weatherInfo.main.humidity }}%
      </p>
      <p style="font-size: 16px; line-height: 1.6">
        💨 <strong>현재 풍속:</strong> {{ weatherInfo.wind.speed }} m/s
      </p>
    </div>

    <button
      @click="router.push('/')"
      style="
        margin-top: 30px;
        padding: 12px 24px;
        background: #1f2937;
        color: #fff;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: bold;
      "
    >
      ← 메인 대시보드로 돌아가기
    </button>
  </div>
</template>
