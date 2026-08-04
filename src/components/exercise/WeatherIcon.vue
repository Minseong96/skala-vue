<!-- src/components/exercise/WeatherIcon.vue -->
<script setup>
import { computed } from 'vue'

const props = defineProps({
  code: {
    type: String,
    default: '01d',
  },
  size: {
    type: [Number, String],
    default: 40,
  },
})

// Normalize code to match icon templates
const iconType = computed(() => {
  if (!props.code) return '01d'
  return props.code.trim()
})
</script>

<template>
  <div class="ios-weather-icon" :style="{ width: `${size}px`, height: `${size}px` }">
    <!-- 01d: Clear Sun (Natural Golden iOS Sunlight) -->
    <svg
      v-if="iconType === '01d'"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <radialGradient id="sun-glow" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#FFE066" />
          <stop offset="65%" stop-color="#FFAB00" />
          <stop offset="100%" stop-color="#FF8800" />
        </radialGradient>
        <filter id="sun-shadow" x="-30%" y="-30%" width="160%" height="160%">
          <feDropShadow dx="0" dy="2" stdDeviation="4" flood-color="#FF9E00" flood-opacity="0.45" />
        </filter>
      </defs>
      <!-- Sun Core -->
      <circle
        cx="32"
        cy="32"
        r="15"
        fill="url(#sun-glow)"
        filter="url(#sun-shadow)"
        class="sun-core-pulse"
      />
      <!-- Sun Rays -->
      <g
        stroke="#FFA800"
        stroke-width="3"
        stroke-linecap="round"
        opacity="0.85"
        class="sun-rays-spin"
      >
        <line x1="32" y1="7" x2="32" y2="12" />
        <line x1="32" y1="52" x2="32" y2="57" />
        <line x1="7" y1="32" x2="12" y2="32" />
        <line x1="52" y1="32" x2="57" y2="32" />
        <line x1="14.3" y1="14.3" x2="17.8" y2="17.8" />
        <line x1="46.2" y1="46.2" x2="49.7" y2="49.7" />
        <line x1="14.3" y1="49.7" x2="17.8" y2="46.2" />
        <line x1="46.2" y1="17.8" x2="49.7" y2="14.3" />
      </g>
    </svg>

    <!-- 01n: Clear Night (Soft Glowing Silver Moon) -->
    <svg
      v-else-if="iconType === '01n'"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <linearGradient id="moon-grad" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" stop-color="#F8FAFC" />
          <stop offset="100%" stop-color="#94A3B8" />
        </linearGradient>
        <filter id="moon-shadow" x="-20%" y="-20%" width="140%" height="140%">
          <feDropShadow dx="0" dy="2" stdDeviation="3" flood-color="#64748B" flood-opacity="0.35" />
        </filter>
      </defs>
      <path
        d="M42 36C42 45.9411 33.9411 54 24 54C19.7845 54 15.922 52.548 12.8719 50.1066C17.5144 52.5807 22.8407 54 28.5 54C42.5833 54 54 42.5833 54 28.5C54 21.0504 50.8143 14.3461 45.7171 9.6384C48.4069 13.0645 50 17.3482 50 22C50 31.9411 41.9411 40 32 40"
        fill="url(#moon-grad)"
        filter="url(#moon-shadow)"
        class="moon-glow-anim"
      />
    </svg>

    <!-- 02d: Few Clouds (Golden Sun + iOS Cloud) -->
    <svg
      v-else-if="iconType === '02d'"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <radialGradient id="sun-back" cx="50%" cy="50%" r="50%">
          <stop offset="0%" stop-color="#FFE066" />
          <stop offset="100%" stop-color="#FFAB00" />
        </radialGradient>
        <linearGradient id="cloud-front" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#FFFFFF" />
          <stop offset="100%" stop-color="#CBD5E1" />
        </linearGradient>
        <filter id="shadow-cloud" x="-20%" y="-20%" width="140%" height="140%">
          <feDropShadow dx="0" dy="3" stdDeviation="3" flood-color="#0F172A" flood-opacity="0.25" />
        </filter>
      </defs>
      <!-- Sun in background -->
      <circle cx="24" cy="22" r="12" fill="url(#sun-back)" class="sun-core-pulse" />
      <g
        stroke="#FFA800"
        stroke-width="2.5"
        stroke-linecap="round"
        opacity="0.75"
        class="sun-rays-spin-sub"
      >
        <line x1="24" y1="5" x2="24" y2="8" />
        <line x1="8" y1="22" x2="11" y2="22" />
        <line x1="12.7" y1="10.7" x2="14.8" y2="12.8" />
        <line x1="35.3" y1="10.7" x2="33.2" y2="12.8" />
      </g>
      <!-- Cloud in foreground -->
      <path
        d="M22 48H46C51.5228 48 56 43.5228 56 38C56 32.7483 51.9427 28.4447 46.7909 28.0437C45.3951 22.285 40.1983 18 34 18C26.8203 18 21 23.8203 21 31C21 31.6841 21.053 32.3558 21.155 33.0101C17.0782 34.0205 14 37.6601 14 42C14 45.3137 17.5817 48 22 48Z"
        fill="url(#cloud-front)"
        filter="url(#shadow-cloud)"
        class="cloud-float-anim"
      />
    </svg>

    <!-- 02n: Few Clouds Night -->
    <svg
      v-else-if="iconType === '02n'"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <linearGradient id="cloud-night" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#E2E8F0" />
          <stop offset="100%" stop-color="#64748B" />
        </linearGradient>
      </defs>
      <path
        d="M26 14C26 21 21 26 14 26C12 26 10.2 25.5 8.6 24.6C10.8 28 14.6 30 19 30C26.7 30 33 23.7 33 16C33 11.6 31 7.8 27.6 5.6C28.5 7.2 29 9 29 11"
        fill="#94A3B8"
        class="moon-glow-anim"
      />
      <path
        d="M22 48H46C51.5228 48 56 43.5228 56 38C56 32.7483 51.9427 28.4447 46.7909 28.0437C45.3951 22.285 40.1983 18 34 18C26.8203 18 21 23.8203 21 31C21 31.6841 21.053 32.3558 21.155 33.0101C17.0782 34.0205 14 37.6601 14 42C14 45.3137 17.5817 48 22 48Z"
        fill="url(#cloud-night)"
        class="cloud-float-anim"
      />
    </svg>

    <!-- 03d, 03n, 04d, 04n: Scattered / Broken Clouds -->
    <svg
      v-else-if="['03d', '03n', '04d', '04n'].includes(iconType)"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <linearGradient id="cloud-full" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#F1F5F9" />
          <stop offset="100%" stop-color="#94A3B8" />
        </linearGradient>
        <filter id="shadow-cloud-full" x="-20%" y="-20%" width="140%" height="140%">
          <feDropShadow dx="0" dy="3" stdDeviation="3" flood-color="#0F172A" flood-opacity="0.3" />
        </filter>
      </defs>
      <path
        d="M20 50H46C52.0751 50 57 45.0751 57 39C57 33.2206 52.5539 28.481 46.8837 28.0427C45.3524 21.691 39.6384 17 32.8 17C24.9023 17 18.5 23.4023 18.5 31.3C18.5 32.0526 18.5583 32.7915 18.6706 33.5115C14.1818 34.6226 10.8 38.626 10.8 43.4C10.8 47.045 14.9213 50 20 50Z"
        fill="url(#cloud-full)"
        filter="url(#shadow-cloud-full)"
        class="cloud-float-anim"
      />
    </svg>

    <!-- 09d, 09n, 10d, 10n: Rain -->
    <svg
      v-else-if="['09d', '09n', '10d', '10n'].includes(iconType)"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <linearGradient id="cloud-rain" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#E2E8F0" />
          <stop offset="100%" stop-color="#64748B" />
        </linearGradient>
        <linearGradient id="drop-grad" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#38BDF8" />
          <stop offset="100%" stop-color="#0284C7" />
        </linearGradient>
      </defs>
      <path
        d="M20 42H46C51.5228 42 56 37.5228 56 32C56 26.7483 51.9427 22.4447 46.7909 22.0437C45.3951 16.285 40.1983 12 34 12C26.8203 12 21 17.8203 21 25C21 25.6841 21.053 26.3558 21.155 27.0101C17.0782 28.0205 14 31.6601 14 36C14 39.3137 17.5817 42 22 42Z"
        fill="url(#cloud-rain)"
        class="cloud-float-anim"
      />
      <!-- Raindrops with 3D drop motion -->
      <rect
        x="22"
        y="46"
        width="3.5"
        height="9"
        rx="1.75"
        fill="url(#drop-grad)"
        transform="rotate(15 23.75 50.5)"
        class="raindrop-anim"
      />
      <rect
        x="32"
        y="47"
        width="3.5"
        height="9"
        rx="1.75"
        fill="url(#drop-grad)"
        transform="rotate(15 33.75 51.5)"
        class="raindrop-anim-delay1"
      />
      <rect
        x="42"
        y="46"
        width="3.5"
        height="9"
        rx="1.75"
        fill="url(#drop-grad)"
        transform="rotate(15 43.75 50.5)"
        class="raindrop-anim-delay2"
      />
    </svg>

    <!-- 11d, 11n: Thunderstorm -->
    <svg
      v-else-if="['11d', '11n'].includes(iconType)"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <defs>
        <linearGradient id="cloud-storm" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" stop-color="#94A3B8" />
          <stop offset="100%" stop-color="#334155" />
        </linearGradient>
      </defs>
      <polygon
        points="32,38 25,50 31,50 28,60 38,46 32,46"
        fill="#FACC15"
        class="lightning-flash"
      />
      <path
        d="M20 40H46C51.5228 40 56 35.5228 56 30C56 24.7483 51.9427 20.4447 46.7909 20.0437C45.3951 14.285 40.1983 10 34 10C26.8203 10 21 15.8203 21 23C21 23.6841 21.053 24.3558 21.155 25.0101C17.0782 26.0205 14 29.6601 14 34C14 37.3137 17.5817 40 22 40Z"
        fill="url(#cloud-storm)"
      />
    </svg>

    <!-- 13d, 13n: Snow -->
    <svg
      v-else-if="['13d', '13n'].includes(iconType)"
      viewBox="0 0 64 64"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M20 42H46C51.5228 42 56 37.5228 56 32C56 26.7483 51.9427 22.4447 46.7909 22.0437C45.3951 16.285 40.1983 12 34 12C26.8203 12 21 17.8203 21 25C21 25.6841 21.053 26.3558 21.155 27.0101C17.0782 28.0205 14 31.6601 14 36C14 39.3137 17.5817 42 22 42Z"
        fill="#CBD5E1"
      />
      <circle cx="23" cy="51" r="2.5" fill="#E2E8F0" class="snowflake-anim" />
      <circle cx="33" cy="54" r="2.5" fill="#E2E8F0" class="snowflake-anim-delay1" />
      <circle cx="43" cy="51" r="2.5" fill="#E2E8F0" class="snowflake-anim-delay2" />
    </svg>

    <!-- Fallback / Default Sun -->
    <svg v-else viewBox="0 0 64 64" fill="none" xmlns="http://www.w3.org/2000/svg">
      <radialGradient id="sun-def" cx="50%" cy="50%" r="50%">
        <stop offset="0%" stop-color="#FFE066" />
        <stop offset="100%" stop-color="#FFAB00" />
      </radialGradient>
      <circle cx="32" cy="32" r="16" fill="url(#sun-def)" class="sun-core-pulse" />
    </svg>
  </div>
</template>

<style scoped>
.ios-weather-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.ios-weather-icon:hover {
  transform: scale(1.14);
}
svg {
  width: 100%;
  height: 100%;
  overflow: visible;
}

/* Animations */
.sun-rays-spin {
  transform-origin: 32px 32px;
  animation: spinRays 28s linear infinite;
}

.sun-rays-spin-sub {
  transform-origin: 24px 22px;
  animation: spinRays 24s linear infinite;
}

@keyframes spinRays {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.sun-core-pulse {
  animation: pulseSun 3.5s ease-in-out infinite alternate;
}

@keyframes pulseSun {
  from {
    transform: scale(1);
    opacity: 0.95;
  }
  to {
    transform: scale(1.06);
    opacity: 1;
  }
}

.cloud-float-anim {
  animation: cloudFloat 4s ease-in-out infinite alternate;
}

@keyframes cloudFloat {
  from {
    transform: translateX(-1.5px);
  }
  to {
    transform: translateX(2.5px);
  }
}

.raindrop-anim {
  animation: raindropFall 1.3s ease-in-out infinite;
}
.raindrop-anim-delay1 {
  animation: raindropFall 1.3s ease-in-out 0.4s infinite;
}
.raindrop-anim-delay2 {
  animation: raindropFall 1.3s ease-in-out 0.8s infinite;
}

@keyframes raindropFall {
  0% {
    transform: translateY(-2px) rotate(15deg);
    opacity: 0.3;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translateY(7px) rotate(15deg);
    opacity: 0.2;
  }
}

.moon-glow-anim {
  animation: moonFloat 4.5s ease-in-out infinite alternate;
}

@keyframes moonFloat {
  from {
    transform: translateY(0);
    filter: drop-shadow(0 2px 4px rgba(148, 163, 184, 0.3));
  }
  to {
    transform: translateY(-2px);
    filter: drop-shadow(0 4px 10px rgba(148, 163, 184, 0.6));
  }
}

.lightning-flash {
  animation: flashLightning 2.2s infinite;
}

@keyframes flashLightning {
  0%,
  80%,
  100% {
    opacity: 0.8;
  }
  85%,
  95% {
    opacity: 1;
    filter: drop-shadow(0 0 10px #facc15);
  }
}

.snowflake-anim {
  animation: snowflakeFall 2s ease-in-out infinite;
}
.snowflake-anim-delay1 {
  animation: snowflakeFall 2s ease-in-out 0.6s infinite;
}
.snowflake-anim-delay2 {
  animation: snowflakeFall 2s ease-in-out 1.2s infinite;
}

@keyframes snowflakeFall {
  0% {
    transform: translateY(-2px);
    opacity: 0.3;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translateY(5px);
    opacity: 0.3;
  }
}
</style>
