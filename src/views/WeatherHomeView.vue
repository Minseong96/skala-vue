<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import BaseDashboardCard from '@/components/exercise/BaseDashboardCard.vue'
import SearchBar from '@/components/exercise/SearchBar.vue'
import WeatheCard from '@/components/exercise/WeatheCard.vue'

const router = useRouter()

// 도시별 위도/경도 정보 (API 호출용)
const citiesMeta = [
  { id: 'city_01', name: '서울특별시', lat: 37.5665, lon: 126.978 },
  { id: 'city_03', name: 'West-lafayette', lat: 40.4259, lon: -86.9081 },
  { id: 'city_04', name: '부산광역시', lat: 35.1796, lon: 129.0756 },
  { id: 'city_05', name: '대구광역시', lat: 35.8714, lon: 128.6014 },
]

const weatherList = ref([]) // API 데이터를 담을 빈 배열
const searchQuery = ref('')
const selectedCityInfo = ref('카드를 클릭하거나 검색해 보세요.')
const isLoading = ref(false)
const errorMessage = ref('')

// 🌟 핵심: 화면이 마운트될 때 모든 도시의 실시간 날씨를 한 번에 가져오기!
const fetchAllWeather = async () => {
  isLoading.value = true
  const API_KEY = '31df255e3ec6209feb7ed06f52a7a1f8'

  try {
    // 병렬로 모든 도시의 API 요청을 보냄
    const requests = citiesMeta.map((city) =>
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.lat}&lon=${city.lon}&appid=${API_KEY}&units=metric&lang=kr`,
      ),
    )
    const responses = await Promise.all(requests)

    // 받아온 응답 데이터를 화면용 형식으로 가공
    weatherList.value = responses.map((response, index) => {
      const data = response.data
      return {
        id: citiesMeta[index].id,
        name: citiesMeta[index].name, // 👈 API가 주는 영어 이름 대신 우리가 지정한 이름 사용!
        temp: Math.round(data.main.temp),
        status: data.weather[0].description,
      }
    })
  } catch (error) {
    console.error('날씨 정보를 가져오지 못했습니다:', error)
    errorMessage.value = '실시간 날씨 정보를 불러오는 데 실패했습니다.'
  } finally {
    isLoading.value = false
  }
}

// 컴포넌트가 처음 렌더링될 때 API 호출 함수 실행
onMounted(() => {
  fetchAllWeather()
})

const filteredWeatherList = computed(() => {
  if (!searchQuery.value) return weatherList.value
  return weatherList.value.filter((city) =>
    city.name.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

const handleSelectCard = (cityName) => {
  selectedCityInfo.value = `${cityName}이 선택되었습니다.`
}

const handleDetail = (cityId) => {
  router.push(`/weather/${cityId}`)
}

const getCardTheme = (status, temp) => {
  if (status.includes('비')) return { bg: '#f0f7ff', border: '#bae6fd' }
  if (status.includes('구름')) return { bg: '#f8fafc', border: '#cbd5e1' }
  if (temp >= 28) return { bg: '#fff5f5', border: '#feb2b2' }
  return { bg: '#fffbeb', border: '#fde68a' }
}
</script>

<template>
  <div style="padding: 20px; max-width: 600px; margin: 0 auto; color: #333">
    <h1 style="color: #fff; text-align: center">🌍 실시간 대시보드</h1>
    <SearchBar v-model="searchQuery" />

    <h3 style="color: #fff; margin-top: 20px; margin-bottom: 10px">지역별 실시간 날씨 현황</h3>

    <!-- 로딩 중일 때 -->
    <p
      v-if="isLoading"
      style="text-align: center; padding: 20px; color: #60a5fa; font-weight: bold"
    >
      🔄 실시간 데이터를 불러오는 중입니다...
    </p>

    <!-- 에러 발생 시 -->
    <p
      v-else-if="errorMessage"
      style="color: #ef4444; font-weight: bold; text-align: center; padding: 20px"
    >
      {{ errorMessage }}
    </p>

    <!-- 데이터 로드 성공 시 -->
    <template v-else>
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
        <WeatheCard :city="city" @select-card="handleSelectCard" @click-detail="handleDetail" />
      </BaseDashboardCard>
    </template>

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
