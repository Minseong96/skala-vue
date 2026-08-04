<script setup>
defineProps({
  bgColor: { type: String, default: 'rgba(15, 23, 42, 0.72)' },
  borderColor: { type: String, default: 'rgba(255, 255, 255, 0.16)' },
})
</script>

<template>
  <div
    :style="{
      backgroundColor: bgColor,
      borderColor: borderColor,
      borderWidth: '1px',
      borderStyle: 'solid',
      padding: '16px',
      borderRadius: '24px',
      boxShadow: '0 18px 38px rgba(2, 6, 23, 0.18)',
      transition: 'transform 0.2s ease, box-shadow 0.2s ease',
      backdropFilter: 'blur(24px)',
      WebkitBackdropFilter: 'blur(24px)',
    }"
    class="dashboard-card"
  >
    <slot></slot>
  </div>
</template>

<style scoped>
.dashboard-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 16px 32px rgba(2, 6, 23, 0.22);
}
</style>
