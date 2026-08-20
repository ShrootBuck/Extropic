<script setup lang="ts">
import NoiseField from './components/NoiseField.vue'
</script>

<template>
  <div class="deck-bg">
    <NoiseField :intensity="1" :embers="30" />
    <div class="scan" />
    <div class="vignette" />
  </div>
</template>

<style scoped>
.deck-bg {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  overflow: hidden;
}
.scan {
  position: absolute; inset: 0;
  background: repeating-linear-gradient(0deg, rgba(255,255,255,0.014) 0 1px, transparent 1px 3px);
  mix-blend-mode: overlay;
}
.vignette {
  position: absolute; inset: 0;
  background: radial-gradient(120% 90% at 50% 45%, transparent 45%, rgba(0,0,0,0.55) 100%);
}
</style>
