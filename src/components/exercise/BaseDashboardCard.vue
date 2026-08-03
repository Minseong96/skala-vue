<script setup>
defineProps({
  bgColor: { type: String, default: '#ffffff' },
  borderColor: { type: String, default: '#e2e8f0' },
})
</script>

<template>
  <div
    :style="{
      backgroundColor: bgColor,
      borderColor: borderColor,
      borderWidth: '1px',
      borderStyle: 'solid',
      padding: '15px',
      marginBottom: '15px',
      borderRadius: '8px',
      boxShadow: '0 2px 4px rgba(0,0,0,0.04)',
      transition: 'transform 0.2s ease, box-shadow 0.2s ease',
    }"
    class="dashboard-card"
  >
    <slot></slot>
  </div>
</template>

<style scoped>
.dashboard-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}
</style>
