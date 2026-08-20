<script setup lang="ts">
import { computed } from 'vue'
import { useNav } from '@slidev/client'

const { currentSlideNo, total } = useNav()
const pct = computed(() => Math.max(0, Math.min(100, (currentSlideNo.value / total.value) * 100)))
</script>

<template>
  <div class="deck-top">
    <div class="prog" :style="{ width: pct + '%' }" />
    <div class="idx mono">
      {{ String(currentSlideNo).padStart(2, '0') }}<span>/{{ String(total).padStart(2, '0') }}</span>
    </div>
  </div>
</template>

<style scoped>
.deck-top { position: absolute; inset: 0; pointer-events: none; z-index: 40; }
.prog {
  position: absolute; top: 0; left: 0; height: 2px;
  background: linear-gradient(90deg, #38bdf8, #a78bfa 45%, #ff6b35);
  box-shadow: 0 0 12px rgba(255,107,53,0.8);
  transition: width 480ms cubic-bezier(.2,.7,.2,1);
}
.idx {
  position: absolute; right: 1.35rem; bottom: 1rem;
  font-size: 0.56rem; letter-spacing: 0.18em; color: rgba(255,255,255,0.30);
}
.idx span { color: rgba(255,255,255,0.14); }
</style>
