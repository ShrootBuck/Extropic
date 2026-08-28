<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

const shown = ref(0)
const counter = ref(0)
let raf = 0

onMounted(() => {
  const t0 = performance.now()
  const tick = () => {
    const e = Math.min(1, (performance.now() - t0) / 1500)
    const k = 1 - Math.pow(1 - e, 3)
    shown.value = k
    counter.value = Math.round(k * 10000)
    if (e < 1) raf = requestAnimationFrame(tick)
  }
  raf = requestAnimationFrame(tick)
  onBeforeUnmount(() => cancelAnimationFrame(raf))
})

// log scale across 5 decades
const pos = (v: number) => Math.log10(v) / 4
</script>

<template>
  <div class="bars">
    <div class="axis mono">
      <span v-for="(t, d) in ['1×','10×','100×','1,000×','10,000×']" :key="d"
            class="tick" :style="{ left: (d/4*100)+'%' }">{{ t }}</span>
    </div>

    <div class="row">
      <div class="name mono">GPU today</div>
      <div class="track">
        <div class="fill gpu" :style="{ width: (pos(10000)*100*shown) + '%' }" />
      </div>
      <div class="tag mono">baseline energy</div>
    </div>

    <div class="row">
      <div class="name mono">TSU <span class="sub">(simulated)</span></div>
      <div class="track">
        <div class="fill tsu" :style="{ width: Math.max(1.2, pos(1.35)*100*shown) + '%' }" />
      </div>
      <div class="tag mono hot">same job, ~1/10,000 the energy</div>
    </div>

    <div class="big">
      <span class="num mono grad-hot">{{ counter.toLocaleString() }}×</span>
      <span class="lab">claimed energy efficiency, on the workloads it was built for</span>
    </div>

    <div class="src mono">
      source &mdash; Extropic's own <em>simulation</em> of a production-scale TSU running a
      Denoising Thermodynamic Model. Not a measured chip.
    </div>
  </div>
</template>

<style scoped>
.bars { display: flex; flex-direction: column; gap: 0.75rem; width: 100%; }
.axis { position: relative; height: 12px; }
.tick { position: absolute; font-size: 0.5rem; color: #46506a; transform: translateX(-50%); letter-spacing: 0.05em; }
.row { display: flex; align-items: center; gap: 0.75rem; }
.name { width: 8.5rem; font-size: 0.78rem; color: #e9ecf4; text-align: right; }
.name .sub { color: #5d6780; font-size: 0.58rem; }
.track { flex: 1; height: 20px; border-radius: 8px; background: rgba(255,255,255,0.05); overflow: hidden; }
.fill { height: 100%; border-radius: 8px; transition: none; }
.gpu { background: linear-gradient(90deg, rgba(120,140,180,0.35), #94a3b8); }
.tsu { background: linear-gradient(90deg, #ffb347, #ff4d1c); box-shadow: 0 0 18px rgba(255,90,40,0.9); }
.tag { width: 12rem; font-size: 0.6rem; color: #5d6780; }
.tag.hot { color: #ffb347; }
.src { align-self: center; max-width: 42rem; margin-top: 0.75rem; font-size: 0.62rem; line-height: 1.5; text-align: center; letter-spacing: 0.04em; color: #5d6780; }
.src em { color: #7d879c; font-style: italic; }
.big { display: flex; align-items: baseline; gap: 0.9rem; margin-top: 0.7rem; padding-left: 9.2rem; }
.num { font-size: 3rem; font-weight: 700; letter-spacing: -0.03em; }
.lab { font-size: 0.74rem; color: #7d879c; }
</style>
