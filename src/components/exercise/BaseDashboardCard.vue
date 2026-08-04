<!-- src/components/exercise/BaseDashboardCard.vue -->
<script setup>
import { ref } from 'vue'

defineProps({
  bgColor: { type: String, default: 'rgba(15, 23, 42, 0.72)' },
  borderColor: { type: String, default: 'rgba(255, 255, 255, 0.16)' },
})

const cardRef = ref(null)
const tiltTransform = ref('perspective(1000px) rotateX(0deg) rotateY(0deg) scale3d(1, 1, 1)')
const isHovered = ref(false)

const handleMouseMove = (e) => {
  if (!cardRef.value) return
  const rect = cardRef.value.getBoundingClientRect()
  const x = e.clientX - rect.left - rect.width / 2
  const y = e.clientY - rect.top - rect.height / 2
  const rotateX = (-(y / (rect.height / 2)) * 8).toFixed(2)
  const rotateY = ((x / (rect.width / 2)) * 8).toFixed(2)
  tiltTransform.value = `perspective(1000px) rotateX(${rotateX}deg) rotateY(${rotateY}deg) scale3d(1.02, 1.02, 1.02)`
}

const handleMouseEnter = () => {
  isHovered.value = true
}

const handleMouseLeave = () => {
  isHovered.value = false
  tiltTransform.value = 'perspective(1000px) rotateX(0deg) rotateY(0deg) scale3d(1, 1, 1)'
}
</script>

<template>
  <div
    ref="cardRef"
    @mousemove="handleMouseMove"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
    :style="{
      backgroundColor: bgColor,
      borderColor: borderColor,
      borderWidth: '1px',
      borderStyle: 'solid',
      padding: '16px',
      borderRadius: '24px',
      boxShadow: isHovered
        ? '0 24px 48px rgba(0, 0, 0, 0.38), 0 0 20px rgba(56, 189, 248, 0.15)'
        : '0 12px 28px rgba(2, 6, 23, 0.2)',
      transform: tiltTransform,
      transition: isHovered
        ? 'transform 0.08s ease-out, box-shadow 0.2s ease'
        : 'transform 0.5s ease, box-shadow 0.5s ease',
      backdropFilter: 'blur(24px)',
      WebkitBackdropFilter: 'blur(24px)',
      transformStyle: 'preserve-3d',
    }"
    class="dashboard-card"
  >
    <div class="card-content-3d">
      <slot></slot>
    </div>
  </div>
</template>

<style scoped>
.dashboard-card {
  will-change: transform;
  cursor: pointer;
}
.card-content-3d {
  transform: translateZ(14px);
  transition: transform 0.2s ease;
}
</style>
