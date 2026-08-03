<script setup>
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

// 임시 Mock Data베이스
const weatherData = {
  city_01: { name: '서울특별시', temp: '28°C', status: '맑음', humidity: '55%', wind: '2.5m/s' },
  city_02: { name: '수원시', temp: '24°C', status: '비', humidity: '80%', wind: '4.1m/s' },
  city_03: {
    name: 'West-lafayette',
    temp: '10°C',
    status: '맑음',
    humidity: '40%',
    wind: '3.0m/s',
  },
  city_04: { name: '부산광역시', temp: '26°C', status: '구름', humidity: '65%', wind: '3.5m/s' },
  city_05: { name: '대구광역시', temp: '31°C', status: '맑음', humidity: '45%', wind: '1.8m/s' },
  city_06: {
    name: '제주특별자치도',
    temp: '25°C',
    status: '구름',
    humidity: '70%',
    wind: '5.2m/s',
  },
}

// 주소창의 :cityId를 읽어와서 데이터 매칭
const currentCity = weatherData[route.params.cityId] || {
  name: '알 수 없는 도시',
  temp: '-',
  status: '-',
  humidity: '-',
  wind: '-',
}
</script>

<template>
  <div
    style="
      padding: 20px;
      max-width: 600px;
      margin: 0 auto;
      background: #fff;
      border-radius: 8px;
      color: #333;
    "
  >
    <h2>지역별 상세 기상 관측 정보</h2>
    <div
      style="
        margin-top: 20px;
        background: #f8fafc;
        padding: 15px;
        border-radius: 6px;
        border: 1px solid #cbd5e1;
      "
    >
      <p>📍 <strong>지점 지역:</strong> 대한민국 {{ currentCity.name }}</p>
      <p>🌡️ <strong>실시간 기온:</strong> {{ currentCity.temp }}</p>
      <p>☁️ <strong>기상 현황:</strong> {{ currentCity.status }}</p>
      <p>💧 <strong>대기 습도:</strong> {{ currentCity.humidity }}</p>
      <p>💨 <strong>현재 풍속:</strong> {{ currentCity.wind }}</p>
    </div>
    <button
      @click="router.push('/')"
      style="
        margin-top: 20px;
        padding: 10px 20px;
        background: #1e293b;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      "
    >
      ← 메인 대시보드로 돌아가기
    </button>
  </div>
</template>
