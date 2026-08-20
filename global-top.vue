<script setup lang="ts">
import { useNav } from '@slidev/client'

const { currentSlideNo, total } = useNav()
</script>

<template>
  <div class="deck-top">
    <div class="idx mono">
      {{ String(currentSlideNo).padStart(2, '0') }}<span>/{{ String(total).padStart(2, '0') }}</span>
    </div>
  </div>
</template>

<style scoped>
.deck-top { position: absolute; inset: 0; pointer-events: none; z-index: 40; }
.idx {
  position: absolute; right: 1.35rem; bottom: 1rem;
  font-size: 0.56rem; letter-spacing: 0.18em; color: rgba(255,255,255,0.30);
}
.idx span { color: rgba(255,255,255,0.14); }
</style>
